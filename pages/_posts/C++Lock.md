---
title: Linux下4种常见进程锁
date: 2020-02-07 10:42:47
tags: 
- linux     
- c++
---

## <font color=#ba3925>1.互斥锁</font>
在编程中，引入了对象互斥锁的概念，来保证共享数据操作的完整性。每个对象都对应于一个可称为" 互斥锁" 的标记，这个标记用来保证在任一时刻，只能一个线程访问该对象。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。
参考　[互斥锁](https://baike.baidu.com/item/互斥锁/841823?fr=aladdin#15)

### 特点：
保证多个线程执行时只有一个线程能够的在同一时间段访问变量或者函数
互斥锁实际的效率还是可以让人接受的，加锁的时间大概100ns左右，而实际上互斥锁的一种可能的实现是先自旋一段时间，当自旋的时间超过阀值之后再将程投入睡眠中，因此在并发运算中使用互斥锁（每次占用锁的时间很短）的效果可能不亚于使用自旋锁。
实现:
- 创建
  - POSIX定义了一个宏 **PTHREAD_MUTEX_INITIALIZER** 来静态初始化互斥锁，方法如下： `pthread_mutex_t mutex=PTHREAD_MUTEX_INITIALIZER;` LinuxThreads实现中，**pthread_mutex_t** 是一个结构，而**PTHREAD_MUTEX_INITIALIZER**则是一个结构常量
  - 动态方式是采用**pthread_mutex_init()**函数来初始化互斥锁
- 销毁
  
  - **pthread_mutex_destroy()**用于注销一个互斥锁
- 锁操作实现
  
  - 锁操作主要包括加锁pthread_mutex_lock()、解锁pthread_mutex_unlock()和测试加锁 pthread_mutex_trylock()三个。
  
### c++实现
```c++
class TLock{
public:
    TLock(void)  {pthread_mutex_init(&m_lock, NULL); }  //初始化所变量
    ~TLock(void) {pthread_mutex_destory(&m_lock);    }  //销毁锁变量 

    void lock(void) { 
        int RetCode = pthread_mutex_lock(&m_lock); 
        if(RetCode) { 
            throw .... 
        } 
    } //未能锁住当前变量　抛异常出去
    void release(void) { 
        int RetCode = pthread_mutex_unlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }//未能释放当前锁变量　抛异常出去
    void trylock(void){
        int RetCode = pthread_mutex_trylock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }
protected:
    mutable pthread_mutex_t m_lock; //定义可变的变量　锁变量
}
```

## <font color=#ba3925>2.自旋锁</font>
自旋锁是专为防止多处理器并发而引入的一种锁，它在内核中大量应用于中断处理等部分（对于单处理器来说，防止中断处理中的并发可简单采用关闭中断的式，即在标志寄存器中关闭/打开中断标志位，不需要自旋锁）
参考 [自旋锁](https://baike.baidu.com/item/自旋锁/9137985?fr=aladdin)

### 特点

- 其实，自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就**一直循环在那里**看是否该自旋锁的保持者已经释放了锁，”自旋”一词就是因此而得名。

- **可能会产生死锁**。试图递归地获得自旋锁必然会引起死锁。

- 过多占用**cpu**资源。如果不加限制，由于申请者一直在循环等待，因此自旋锁在锁定的时候,如果不成功,不会睡眠,会持续的尝试,单**cpu**的时候自旋锁会让其process动不了. 因此，一般自旋锁实现会有一个参数限定最多持续尝试次数. 超出后, 自旋锁放弃当前time slice. 等下一次机会。

- 自旋锁比较适用于锁使用者保持锁时间比较短的情况。正是由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率高于互斥锁。

### 实现：
  - 创建/销毁
    - 创建：pthread_spin_init(&mr_lock,0);
    - 销毁：pthread_spin_destory(&mr_lock);
    - 申请锁：pthread_spin_lock(&mr_lock);
    - 释放锁：pthread_spin_unlock(&mr_lock);

### c++实现

```c++
class TSpinLock{
    TSpinLock(void)  {pthread_spin_init(&m_lock, 0); }  //初始化所变量
    ~TSpinLock(void) {pthread_spin_destory(&m_lock);    }  //销毁锁变量 
    void lock(void) { 
        int RetCode = pthread_spin_lock(&m_lock); 
        if(RetCode) { 
            throw .... 
        } 
    } //未能锁住当前变量　抛异常出去
    void release(void) { 
        int RetCode = pthread_spin_unlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }//未能释放当前锁变量　抛异常出去
    bool trylock(void){
        int RetCode = pthread_spin_trylock(&m_lock); 
        if(RetCode == 0){
            return 0;
        }else if(RetCode == EBUSY){
            throw ....
        }
        return false;
    }
protected:
    mutable pthread_spin_t m_lock; //定义可变的变量　锁变量
};
```

## <font color=#ba3925>3.读写锁</font>
读写锁是计算机程序的并发控制的一种同步机制，也称**共享-互斥锁**、多读者-单写者锁，它是一种特殊的自旋锁。 读写锁是用于解决类读写问题，比如有些公共数据修改的机会很少，但其读的机会很多。并且在读的过程中会伴随着查找，给这种代码加锁会降低我们的程序率，读写锁可以解决此类问题；其中读操作可并发重入，而写操作是互斥的。
参考 [读写锁](https://blog.csdn.net/c_base_jin/article/details/90246452)

### 特点

- 读者优先策略：允许最大并发，但写操作可能饿死
  - 实现方式　
    读者到：
    1）无读者、写者，新读者可以读
    2）有写者等，但有其它读者正在读，则新读者也可以读
    3）有写者写，新读者等
    写者到：
    1）无读者，新写者可以写
    2）有读者，新写者等待
    3）有其它写者，新写者等待
- 写者优先：一旦所有已经开始的读操作完成，等待的写操作立即获得锁。
  - 实现方式
    读者到：
    1）无读者且无写者，新读者读
    2）有读者但无写者等待，新读者读
    3）有读者但有写着等待，读者等待
    4）有写者在写，新读者等待
    写者到：
    1）无读者且无写者，新写者写
    2）有读者在读，新写者等待
    3）有写者写，新写者等待

### c++实现

```c++
//基类实现　ReadLock WriteLock
class TRWLock{
    TRWLock(void)  {pthread_rwlock_init(&m_lock, NULL); }  //初始化所变量
    ~TRWLock(void) {pthread_rwlock_destory(&m_lock);    }  //销毁锁变量 
    void readLock(void){
        int RetCode = pthread_rwlock_rdlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }
    void writeLock(void){
        int RetCode = pthread_rwlock_wrlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }
    void release(void) { 
        int RetCode = pthread_rwlock_unlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }//未能释放当前锁变量　抛异常出去
    void tryReadlock(void){
        int RetCode = pthread_rwlock_tryrdlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }
    //未能释放当前锁变量　抛异常出去
    void tryWritelock(void){
        int RetCode = pthread_rwlock_trywrlock(&m_lock); 
        if(RetCode){
            throw ....
        }
    }
protected:
    mutable pthread_rwlock_t m_lock; //定义可变的变量　锁变量
};
```

## <font color=#ba3925>4.条件锁</font>

条件锁就是所谓的条件变量，某一个线程因为某个条件为满足时可以使用条件变量使改程序处于阻塞状态。一旦条件满足以“信号量”的方式唤醒一个因为该条件被阻塞的线程。最为常见就是在线程池中，起初没有任务时任务队列为空，此时线程池中的线程因为“任务队列为空”这个条件处于阻塞状态。一旦有任务进来，会以信号量的方式唤醒一个线程来处理这个任务。这个过程中就使用到了条件变量__pthread_cond_t__。

常见下列函数进行调用

```c++
pthread_cond_init(pthread_cond_t * condtion, const phtread_condattr_t * condattr);//对条件变量进行动态初始化，相当于new创建对象
pthread_cond_destory(pthread_cond_t * condition);//释放动态申请的条件变量，相当于delete释放对象
pthread_cond_t condition = PTHREAD_COND_INITIALIZER;//静态初始化条件变量
pthread_cond_wait(pthread_cond_t * cond, pthread_mutex_t * mutex);//该函数以阻塞方式执行。如果某个线程中的程序执行了该函数，那么这个线程会以阻塞方式等待，直到收到pthread_cond_signal或者pthread_cond_broadcast函数发来的信号而被唤醒
```

### 用代码进行简易封装(互斥锁，读写锁)

```c++
template<class type_lock>
class TAutoLockImpl{
public:
    TAutoLockImpl(type_lock &lock):m_lock(&lock){
        m_lock->lock();
    }
    void release(void){
        m_lock->release();
        m_lock=nullptr;
    }
    ~TAutoLockImpl(){
        release();
    }
protected:
    type_lock *m_lock;
};
template<class type_lock>
class TAutoReadLockImpl{
public:
    TAutoReadLockImpl(type_lock &lock):m_lock(&lock){
        m_lock->readLock();
    }
    void release(void){
        m_lock->release();
        m_lock = nullptr;
    } 
    ~TAutoReadLockImpl(){
        release();
    }
protected:
    type_lock　*m_lock;
};
template<class type_lock>
class TAutoWriteLockImpl{
public:
    TAutoWriteLockImpl(type_lock &lock):m_lock(&lock){
        m_lock->writeLock();
    }
    void release(void){
        m_lock->release();
        m_lock = nullptr;
    } 
    ~TAutoWriteLockImpl(){
        release();
    }
protected:
    type_lock　*m_lock;
};
typedef TAutoReadLockImpl<TRWLock> TAutoReadLock;
typedef TAutoWriteLockImpl<TRWLock> TAutoWriteLock;
typedef TAutoLockImpl<TLock> TAutoLock;

//调用方式
std::map<std::string, std::string> GolbalMap;//公用资源
static TLock gLock;
//函数为多线程调用
/** 需要遵循以下规则
 * 1.锁的粒度越小越好，使用完则释放
 * 2.避免调用双重锁，找出死锁
 * 3.能用互斥锁绝不用读写锁，性能需要考虑。
 */
void func1(){
    ...
    TAutoLock lock(gLock);
    GobalMap.insert(std::make_pair<std::string,std::string>("a","b"));
    lock.release();
    ...
}

```


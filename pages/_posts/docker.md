---
title: Docker
date: 2020-02-07 10:42:47
tags: Docker
---

## 基本概念
### 1.镜像
Docker镜像就是一个只读的模板，可以用于Docker容器的创建。

### 2.容器
Docker容器是Docker镜像的一个运行实例。它可以被启动、开始、停止、删除。每一个容器都是相互隔离的，而每一个容器的资源隔离所采用的的基础的是cgroup。(__cgroup 的功能在于将一台计算机上的资源（CPU，memory，network）进行分片，来防止进程间不利的资源抢占__)

### 3.仓库
仓库就是集中放置镜像文件的地方。而且针对相同的镜像有着不同的tag作为版本release区分，用户可以根据自己的需求下载对应的tag的镜像。

### 4.注意事项

docker的对于内核的版本的要求也是相应限制的，建议使用Ubuntu作为物理机搭建docker(__Ubuntu内核相较于CentOS新，相应的内核问题就没有那么突出__)，而对于CentOS, 7的话内核选用3.10, 而CentOS6默认内核为2.6，需要升级内核才能适配docker的全部功能。

## Docker 与传统虚拟化技术的对比

![1.jpeg](http://dockone.io/uploads/article/20190709/5e800c09dafefc8c9f38df1c24a049a5.jpeg)

对于传统的虚拟化技术，docker表现出更<font color=green>轻量，更加方便和快捷</font>，能在秒级启动一个容器，能完成传统虚拟化技术的绝大多数功能。虚拟机实现资源隔离的方法是利用一个独立的Guest OS，并利用Hypervisor虚拟化CPU、内存、IO设备等实现的。例如，为了虚拟化内存，Hypervisor会创建一个shadow page table，正常情况下，一个page table可以用来实现从虚拟内存到物理内存的翻译。相比虚拟机实现资源和环境隔离的方案，Docker就显得<font color=red>简练很多</font>，它不像虚拟机一样重新加载一个操作系统内核，引导、加载操作系统内核是一个比较耗时而又消耗资源的过程，Docker是利用Linux内核特性实现的隔离，运行容器的速度几乎等同于直接启动进程。

### docker特性

+ 使用linux中的内核特性 namespace完成系统隔离。
+ 使用linux中的内核特性 cgroup完成每一个容器的资源隔离。用以限制一个进程组能够使用的资源上限，包括CPU, 内存、硬盘、网络。
+ 使用rootfs挂在宿主机的文件挂载到容器内部，把相关数据通过此等方式传递出容器。

### 两者区别

| 类别         | Docker                         | OpenStack                        |
| :----------- | ------------------------------ | -------------------------------- |
| 部署难易程度 | 非常简单                       | 组件多，部署复杂                 |
| 启动速度     | 秒级                           | 分钟级                           |
| 执行性能     | 相当于一个OS中的进程，启动很快 | vm会占用一些宿主机资源           |
| 镜像体积     | 镜像MB级别(官方)               | 虚拟机镜像GB级别                 |
| 隔离性       | 隔离性高                       | 不属于宿主机，为完全隔离         |
| 可管理性     | 单进程                         | 一个独立与宿主机的管理系统(完备) |
| 网络连接     | 较弱                           | 借助neutron可以灵活进行网络管理  |

### Docker隔离的重要技术

​		docker本质就是宿主机的一个进程，docker是通过namespace实现资源隔离，通过cgroup实现资源限制，通过写时复制技术（copy-on-write）实现了高效的文件操作（类似虚拟机的磁盘比如分配500g并不是实际占用物理磁盘500g）

1. <font color=red>`namespace`</font>名字空间

   主要包含6种隔离方式：

   + UTS 使用CLONE_NEWUTS 隔离主机和域名
   + IPC  使用CLONE_NEWWIPC 隔离信号量、消息队列以及共享内存。进程通信方式
   + PID  使用CLONE_NEWPID 隔离进程
   + NETWORK 使用CLONE_NEWNET 隔离网络设备、网络栈和端口等
   + MOUNT 使用CLONE_NEWNS 隔离文件系统的挂载点
   + USER 使用CLONE_NEWUSER 隔离用户和用户组。

2. <font color=red>`Control Group(cgroup)`</font>控制隔离

   cgroup的api以一个伪文件系统的实现方式，用户的程序可以通过文件系统实现cgroup的组件管理
   cgroup的组件管理操作单元可以细粒度到线程级别，另外用户可以创建和销毁cgroup，从而实现资源载分配和再利用
   所有资源管理的功能都以子系统的方式实现，接口统一子任务创建之初与其父任务处于同一个cgroup的控制组

   四大功能：　　　　　　　　

   资源限制：可以对任务使用的资源总额进行限制
   优先级分配：通过分配的cpu时间片数量以及磁盘IO带宽大小，实际上相当于控制了任务运行优先级
   资源统计：可以统计系统的资源使用量，如cpu时长，内存用量等
   任务控制：cgroup可以对任务执行挂起、恢复等操作

3. 

## Docker踩坑（CentOS)

### 1.报错 __kernel:unregister_netdevice: waiting for lo to become free. Usage count = 1__

此报错为因为宿主机内核版本过低，容易出现docker容器中的程序出现网络断开的情况，这个是linux内核中已知的问题[github Issue](https://github.com/moby/moby/issues/5618)

解决方案：

+ 临时解决：重启docker容器。

+ 永久解决：升级宿主机的内核版本，建议升级至最新的稳定版本即可，推荐 linux 4.4 以上，升级步骤如下所示

  - 1.导入公钥

    ```shell
    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 
    ```

  - 2.安装7.x 版本

    ```shell
    rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
    ```

  - 3.安装内核版本

    ```shell
    yum --enablerepo=elrepo-kernel install kernel-lt -y
    ```

  - 4.修改grub引导文件

    Centos7.x 内核升级完毕后，需要修改内核的启动顺序：

    ```shell
    vim /etc/default/grub
    
    GRUB_TIMEOUT=5
    GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
    GRUB_DEFAULT=saved      #把这里的saved改成0
    GRUB_DISABLE_SUBMENU=true
    GRUB_TERMINAL_OUTPUT="console"
    GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0"
    GRUB_DISABLE_RECOVERY="true"
    ```

    接下来还需要运行grub2-mkconfig命令来重新创建内核配置，命令是grub2-mkconfig -o /boot/grub2/grub.cfg，如下：

    ```shell
    grub2-mkconfig -o /boot/grub2/grub.cfg
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-4.17.171-1.el7.elrepo.x86_64
    Found initrd image: /boot/initramfs-4.17.171-1.el7.elrepo.x86_64.img
    Found linux image: /boot/vmlinuz-3.10.0-693.2.2.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-693.2.2.el7.x86_64.img
    Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
    Found linux image: /boot/vmlinuz-0-rescue-f0f31005fb5a436d88e3c6cbf54e25aa
    Found initrd image: /boot/initramfs-0-rescue-f0f31005fb5a436d88e3c6cbf54e25aa.img
    done
    ```

  - 5.重启服务器

    ```shell
    reboot
    ```

  - 6.查看内核版本

    ```shell
    ➜  ~ uname -r
    4.4.209-1.el7.elrepo.x86_64
    ```

    以上情况表示升级内核成功

    参考地址 [参考地址](https://www.cnblogs.com/clsn/p/10925653.html)

### 2.报错 __容器中使用python的asyncio做项目时，很容易程序因为网络问题而挂死__

eg:使用python的async异步方法连接redis，kafka等其他服务器组件的时候。在处理网络异常的时候需要使得抓取异常的粒度更小化。不然一旦所有协程都会因为异常触发二次异常。如下日志所示

```pseudocode
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
[ERROR-10:26:27] – Error on consume Kafka message:
```

所以为了避免这一个问题，需要如下处理：

１.对于网络交互层面，aioredis,aiokafka 两个类库都有断线重连的机制，所以无须考虑其如何保证断开之后的重连的动作。
２.对于单方面网络交互，如goldfish连接不上kafka,redis的，需要在异常处理的时候需要保证粒度更小，避免宽泛的捕获异常造成二次异常抛出。 [asyncio 异常官方文档](https://docs.python.org/zh-cn/3.7/library/asyncio-exceptions.html#asyncio.CancelledError)

如:
应当避免如下操作

```python
try:
     return await redis_cli.get(key)
except Exception:
     logger.warning("catch exception in redis.get")
     return None
```

因为容器内部使用了async的异步io库,所以在捕获异常层面需要对<font color=red>asyncio.CancelledError</font>进行特殊处理，需要做到如下处理

```python
try:
     return await redis_cli.get(key)
execpt asyncio.CancelledError:
     logger.warning("catch exception in redis.get")
     return None
except Exception:
     logger.warning("catch exception in redis.get")
     return None
```

​	3.即使不会在抛出清一色的Error on Consume Kafka messages,但是因为当前对redis的数据是一个强的依赖模块，所以<font color=red>监控内部容器主机链接redis主机的可用性</font>是优先的，是根本上解决问题的<font color=green>关键</font>。虽然异常处理可以在一定程度上减轻挂死的现象，但是改善效果不如提高redis服务的高可用好。


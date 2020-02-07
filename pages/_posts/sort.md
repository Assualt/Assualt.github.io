---
title: 排序算法
date: 2020-02-07 10:42:47
tags: 
- c++
---

## 算法分类

主要分为两大类，分别是比较类排序和非比较类排序。而比较类排序主要分为`冒泡排序`,
`快速排序`,`插入排序`,`希尔排序`,`选择排序`,`堆排序`,`归并排序`。而非比较排序主要分为`计数排序`,`橘排序`,`基数排序`。

- 比较类排序：通过比较来决定元素间的相对次序，由于其时间复杂度不能突破O(nlogn)，因此也称为非线性时间比较类排序。
- 非比较类排序：不通过比较来决定元素间的相对次序，它可以突破基于比较排序的时间下界，以线性时间运行，因此也称为线性时间非比较类排序。

## 时间复杂度比较

| 排序方式 | 时间复杂度(平均) | 时间复杂度(好) | 时间复杂度(坏) | 空间复杂度 | 稳定性 |
| -------- | ---------------- | -------------- | -------------- | ---------- | ------ |
| 插入排序 | O(n2)            | O(n)           | O(n2)          | O(1)       | 稳定   |
| 希尔排序 | O(n1.3)          | O(n)           | O(n2)          | O(1)       | 不稳定 |
| 选择排序 | O(n2)            | O(n2)          | O(n2)          | O(1)       | 不稳定 |
| 堆排序   | O(nlog2n)        | O(nlog2n)      | O(nlog2n)      | O(1)       | 不稳定 |
| 冒泡排序 | O(n2)            | O(n)           | O(n2)          | O(1)       | 稳定   |
| 快速排序 | O(nlog2n)        | O(nlog2n)      | O(n2)          | O(nlog2n)  | 不稳定 |
| 归并排序 | O(nlog2n)        | O(nlog2n)      | O(nlog2n)      | O(n)       | 稳定   |
| -        | -                | -              | -              | -          | -      |
| 计数排序 | O(n+k)           | O(n+k)         | O(n+k)         | O(n+k)     | 稳定   |
| 桶排序   | O(n+k)           | O(n)           | O(n2)          | O(n+k)     | 稳定   |
| 基数排序 | O(n*k)           | O(n*k)         | O(n*k)         | O(n+k)     | 稳定   |

## 排序相关概念

- 稳定：如果a原本在b前面，而a=b，排序之后a仍然在b的前面。

- 不稳定：如果a原本在b的前面，而a=b，排序之后 a 可能会出现在 b 的后面。

- 时间复杂度：对排序数据的总的操作次数。反映当n变化时，操作次数呈现什么规律。

- 空间复杂度：是指算法在计算机。

## 1.冒泡排序

### 简介

  冒泡排序是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

### 描述

  - 1.比较相邻元素,如果前者小于后者,交换前后数据。
  - 2.对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
  - 3.针对所有的元素重复以上的步骤，除了最后一个；
  - 4.重复步骤1~3，直到排序完成。

### 动图显示

  ![冒泡排序](bubblesort.gif)

### 代码c++

  ```c++
  std::vector<int>& bubbleSort(std::vector<int>& arr){
      for(size_t i = 0;i<arr.size()-1; i++){
          for(size_t j = i+1; j < arr.size(); j++>){
              if(arr[i] > arr[j]){
                  int temp = a[i];
                  a[i] = a[j];
                  a[j] = temp;
              }
          }
      }
      return arr;
  }
  ```

  ## 2.选择排序

### 简介

  选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

### 算法描述

  - 1.初始状态：无序区为R[1..n]，有序区为空；
  - 2.第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
  - 3.n-1趟结束，数组有序化了。

### 动图显示

  ![选择排序(selectsort.gif)

### 代码C++

  ```c++
  std::vector<int>& selectorSort(std::vector<int>& arr){
      int len = arr.size();
      int minIndex, temp;
      for (int i = 0; i < len - 1; i++) {
          minIndex = i;
          for (int j = i + 1; j < len; j++) {
              if (arr[j] < arr[minIndex]) {     // 寻找最小的数
                  minIndex = j;                 // 将最小数的索引保存
              }
          }
          temp = arr[i];
          arr[i] = arr[minIndex];
          arr[minIndex] = temp;
      }
      return arr;
  }
  ```

## 3.快速排序

### 简介

  快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。

### 算法描述

  - 1.从数列中挑出一个元素，称为 “基准”（pivot）；

  - 2.重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；

  - 3.递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序

### 动图显示

    ![快速排序](quicksort.gif)


### 代码C++

```c++
  void quickSort(std::vector<int>& arry, int left, int right){
      int i , j , temp, t;
      if(left >right)
          return;
      temp = arry[left];
      i = left;
      j = right;
      while(i!=j){
          while(arry[j] >= tmep && i < j)
              j--;
          while(arry[i] <= temp && i < j)
              i++;
          if(i < j){
              t = arry[i];
              arry[i] = arry[j];
              arry[j] = t;
          }
      }
      arry[left] = arry[i];
      arry[i] = temp;
      quickSort(arry, left, i-1);
      quickSort(arry, i+1, right);
  }
  //调用方式
  quickSort(arry,1,arry.size()-1);
```

## 4.插入排序

### 简介

插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

### 算法描述

- 1.从第一个元素开始，默认当前元素已经排过序。
  
- 2.取出后一个元素，在排过序队列中从后向前搜索。
  
- 3.如果该元素大于新元素，将元素移到下一个位置。
  
- 4.重复步骤3，直到找到已排序的元素小于或者等于新元素的位置。
  
- 5.将新元素插入到该位置后面。
  
- 6.重复步骤2~5
  
### 动图展示

  ![插入排序](insertsort.gif)

### 代码C++

```c++
    void insertsort(std::vecor<int> &arr){
        int minpos=0;
        for(int i = 1; i < arr.size(); i++){
            int key = arr[i];
            int j = i - 1;
            // 0 1 2 3 4 .... j i;
            while(j>=0 && arr[j] > key)
            {
                arr[j+1] = arr[j];
                j--;
            }
            arr[j+1] = key;
      }
    }
```

## 5.希尔排序

### 简介

1959年Shell发明，第一个突破O(n2)的排序算法，是简单插入排序的改进版。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫缩小增量排序。先将整个待排记录序列分割成为若干子序列分别进行直接插入排序,待整个序列中的记录基本有序时再对全体记录进行一次直接插入排序。

### 算法描述

- 1.选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
  
  - 2.按增量序列个数k，对序列进行k 趟排序；
  
  - 3.每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

### 动图显示

![希尔排序](shellsort.gif)

### 代码C++

```c++
void shellsort(std::vector<int> &arr){
    int h  = 1;
    while(h< cp.size() / 3){//动态定义间隔序列
        h = 3 * h + 1;
    }
    while(h>=1){
        for(int i = h; i < arr.size(); i++){
            // j/h h+1 h+2 
            for(int j = i ; j >=h && (cp[j] <  cp[j-h]); j-=h){
                int temp = cp[j];
                cp[j] = cp[j-h];
                cp[j-h] = temp;
            }
        }
    }
}
```

## 6.归并排序

### 简介

  归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。

  ### 算法描述

  - 1.把长度为n的输入序列分成两个长度为n/2的子序列；

  - 2.对这两个子序列分别采用归并排序；

  - 3.将两个排序好的子序列合并成一个最终的排序序列。

### 动图显示

![归并排序](mergesort.gif)

### 代码C++

```c++
    void mergeSort(std::vector<int> &arry,int L,int R){
        if(arry.size()<2)
            return;
        sortprocess(arry,L,R);
    }
    void sortprocess(std::vector<int> &arr,int L,int R){
        if(L<R){
            int mid = L + ((R-L)>>1);  //  (L+R)/2
    		sortprocess(arr,L,mid);
    		sortprocess(arr,mid+1,R);
    		merge(arr,L,mid,R);
        }
    }
    void merge(vector<int> &arr,int L,int mid,int R){
    	std::vector<int> help(R-L+1,0);
    	int p1=L,p2=mid+1,i=0;
    	while(p1<=mid && p2<=R){
    		help[i++] = arr[p1]>arr[p2] ? arr[p2++] : arr[p1++];
    	}
    	while(p1<=mid)
    		help[i++] = arr[p1++];
    	while(p2<=R)
    		help[i++] = arr[p2++];
    	for (int i=0;i<R-L+1;i++){
    		arr[L+i] = help[i];
    	}
    }
```

## 7.计数排序

### 简介

  计数排序不是基于比较的排序算法，其核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

### 算法描述

  - 找出待排序的数组中最大和最小的元素；
  - 统计数组中每个值为i的元素出现的次数，存入数组C的第i项；
  - 对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）；
  - 反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1。

### 动图展示

![计数排序](countsort.gif)

### 代码实现

```c++
void countsort(std::vector<int> &arry){
    if(arry.empty())
        return;
    std::vector<int> cp;
    int maxVal = arry[0];
    for(size_t i = 0; i < arry.size(); i++)
        maxVal = maxVal < arry[i] ? arry[i] : maxVal;
	std::vector<int> newArry(maxVal + 1,0);
    for(size_t i = 0; i < arry.size(); i++)
        newArry[arry[i]] ++;
    for(size_t j = 0; j < newArry.size(); j++)
        while(newArry[j]-->0)
            cp.push_back(j);
    arry.clear();
    arry.swap(cp);
}
```

   计数排序是一个稳定的排序算法。当输入的元素是 n 个 0到 k 之间的整数时，时间复杂度是O(n+k)，空间复杂度也是O(n+k)，其排序速度快于任何比较排序算法。当k不是很大并且序列比较集中时，计数排序是一个很有效的排序算法。

## 8.堆排序

### 简介

  堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

### 算法描述

  - 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
  - 将堆顶元素R[1]与最后一个元素R[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R[1,2…n-1]<=R[n]；
  - 由于交换后新的堆顶R[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R[1]与无序区最后一个元素交换，得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

  ### 动图展示

  ![堆排序](heapsort.gif)

  ![堆的二叉树结构图](heap.png)

### 代码实现

```c++
//调整堆的位置
void heapify(std::vector<int>& arry, int i, int len) {
	int left = 2 * i + 1;
	int right = 2 * i + 2;
	int largest = i;
	//判断左孩子是否大于要比较的当前largest节点
	//大于则交换对应的位置
	if (left < len && arry[left] > arry[largest])
		largest = left;
	//判断有孩子节点是否大于当前的largest节点
	//大于则交换对应的位置
	if (right < len && arry[right] > arry[largest])
		largest = right;
	//如果已经符合堆的特性
	//最大堆：1.根的值大于左右子树的值   2.子树也是最大堆
	if (largest != i) {
		//交换i和largest的值
		swap(arry, i, largest);
		heapify(arry, largest, len); //递归处理后续的
	}
}
//堆是一棵完全二叉树,所以可以利用完全二叉树的性质得到相应的父子节点的位置信息
//实现使用最大堆实现排序功能
void heapsort1(std::vector<int>& arry) {
	int size = arry.size();
	// 构建大根堆（从最后一个非叶子节点向上）
	for (int i = size / 2-1; i >= 0; i--) {
		heapify(arry, i, size);
	}
	for (size_t i = arry.size() - 1; i > 0; i--) {
		swap(arry, 0, i);
		heapify(arry, 0, i); //将未完成的堆进行再次排序
	}
}
void swap(vector<int>& arr, int s, int d) {
	int temp = arr[s];
	arr[s] = arr[d];
	arr[d] = temp;
}
```

## 9.基数排序

### 简介

  基数排序是按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。有时候有些属性是有优先级顺序的，先按低优先级排序，再按高优先级排序。最后的次序就是高优先级高的在前，高优先级相同的低优先级高的在前。

### 算法描述

  - 取得数组中的最大数，并取得位数；
  - arr为原始数组，从最低位开始取每个位组成radix数组；
  - 对radix进行计数排序（利用计数排序适用于小范围数的特点）；

### 动图展示

![基数排序](radixsort.gif)

### 代码实现

  基数排序基于分别排序，分别收集，所以是稳定的。但基数排序的性能比桶排序要略差，每一次关键字的桶分配都需要O(n)的时间复杂度，而且分配之后得到新的关键字序列又需要O(n)的时间复杂度。假如待排数据可以分为d个关键字，则基数排序的时间复杂度将是O(d*2n) ，当然d要远远小于n，因此基本上还是线性级别的。

  基数排序的空间复杂度为O(n+k)，其中k为桶的数量。一般来说n>>k，因此额外空间需要大概n个左右。
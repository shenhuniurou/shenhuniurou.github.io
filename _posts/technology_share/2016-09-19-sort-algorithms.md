---
layout: post
title: 几种经典排序算法
category: blog
tags: 排序、算法
---

排序算法是非常常见也非常基础的算法，虽然已经可以很方便的使用，但是理解排序算法可以帮助我们找到解题的方向。

## 1. 冒泡排序 (Bubble Sort)

冒泡排序是最简单粗暴的排序方法之一。它的原理很简单，每次从左到右两两比较，把大的交换到后面，每次可以确保将前M个元素的最大值移动到最右边。

**步骤**

1. 从左开始比较相邻的两个元素x和y，如果 x > y 就交换两者
2. 执行比较和交换，直到到达数组的最后一个元素
3. 重复执行1和2，直到执行n次，也就是n个最大元素都排到了最后

Java实现方法：

```java
/**
 * 冒泡排序
 * 比较相邻的元素。如果第一个比第二个大，就交换他们两个。  
 * 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。  
 * 针对所有的元素重复以上的步骤，除了最后一个。
 * 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。 
 * @param numbers 需要排序的整型数组
 */
public static void bubbleSort(int[] numbers) {
    int temp = 0;
    int size = numbers.length;
    for (int i = 0 ; i < size - 1; i++) {
	    for (int j = 0 ;j < size - i - 1; j++) {
	        if (numbers[j] > numbers[j + 1]) { 
	        	//交换两数位置
		        temp = numbers[j];
		        numbers[j] = numbers[j + 1];
		        numbers[j + 1] = temp;
	        }
	    }
    }
}
```

交换的那一步可以不借助temp，方法是

```java
numbers[j] += numbers[j + 1];
numbers[j + 1] = numbers[j] - numbers[j + 1];
numbers[j] -= numbers[j + 1];
```

**复杂度分析**

由于我们要重复执行n次冒泡，每次冒泡要执行n次比较（实际是1到n的等差数列，也就是`(a1 + an) * n / 2`），也就是 `O(n^2)`。 空间复杂度是`O(n)`。



## 2. 选择排序（Selection Sort）

选择排序的原理是，每次都从乱序数组中找到最大（最小）值，放到当前乱序数组头部，最终使数组有序。

**步骤**

1. 从左开始，选择后面元素中最小值，和最左元素交换

2. 从当前已交换位置往后执行，直到最后一个元素

```java
/**
 * 选择排序算法
 * 在未排序序列中找到最小元素，存放到排序序列的起始位置
 * 再从剩余未排序元素中继续寻找最小元素，然后放到排序序列末尾。
 * 以此类推，直到所有元素均排序完毕。
 * @param numbers
 */
public void selectSort(int[] numbers) {
    int size = numbers.length; //数组长度
    int temp; //中间变量
    for (int i = 0; i < size; i++) {
        int min = i;   //待确定的位置
        //选择出应该在第i个位置的数
        for (int j = size - 1; j > i; j--) {// 这里或者是for (int j = i + 1; j < size; j++)
            if (numbers[j] < numbers[min]) {
                min = j;
            }
        }

        //交换两个数
        temp = numbers[i];
        numbers[i] = numbers[min];
        numbers[min] = temp;
    }
}
```

**复杂度分析**

每次要找一遍最小值，最坏情况下找n次，这样的过程要执行n次，所以时间复杂度还是`O(n^2)`。空间复杂度是`O(n)`。



## 3. 插入排序（Insertion Sort）

插入排序的原理是从左到右，把选出的一个数和前面的数进行比较，找到最适合它的位置放入，使前面部分有序。

**步骤**

1. 从左开始，选出当前位置的数x，和它之前的数y比较，如果x < y则交换两者
2. 对x之前的数都执行1步骤，直到前面的数字都有序
3. 选择有序部分后一个数字，插入到前面有序部分，直到没有数字可选择

Java实现方法：

```java
/**  
 * 插入排序
 * 
 * 从第一个元素开始，该元素可以认为已经被排序
 * 取出下一个元素，在已经排序的元素序列中从后向前扫描 
 * 如果该元素（已排序）大于新元素，将该元素移到下一位置  
 * 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置  
 * 将新元素插入到该位置中  
 * 重复步骤2  
 * @param numbers  待排序数组
 */  
public static void insertSort(int[] numbers) {
	int size = numbers.length;
    int temp, j;
    for (int i = 0; i < numbers.length; i++) {
        for (int j = i; j > 0; j--) {
            if (numbers[j] < numbers[j - 1]) {
                int temp = numbers[j];
                numbers[j] = numbers[j - 1];
                numbers[j - 1] = temp;
            }
        }
    }
}
```

**复杂度分析**

因为要选择n次，而且插入时最坏要比较n次，所以时间复杂度同样是`O(n^2)`。空间复杂度是`O(n)`。


## 4. 希尔排序（Shell Sort）

希尔排序从名字上看不出来特点，因为它是以发明者命名的。它的另一个名字是“递减增量排序算法“。这个算法可以看作是插入排序的优化版，因为插入排序需要一位一位比较，然后放置到正确位置。为了提升比较的跨度，希尔排序将数组按照一定步长分成几个子数组进行排序，通过逐渐减短步长来完成最终排序。

**步骤**

1. 计算当前步长，按步长划分子数组
2. 子数组内插入排序
3. 步长除以2后继续1、2两步，直到步长最后变成1


```java
public static void shellSort(int[] data) {
    //每次将步长缩短为原来的一半
    for (int increment = data.length / 2; increment > 0; increment /= 2) {
        for (int i = increment; i < data.length; i++) {
            int j = i;
            while (j - increment >= 0 && data[j] < data[j - increment]) {
                //插入排序采用交换法
                swap(data, j, j - increment);
                j -= increment;
            }
        }
    }
}

public static void swap(int[] arr, int a, int b) {
    arr[a] = arr[a] + arr[b];
    arr[b] = arr[a] - arr[b];
    arr[a] = arr[a] - arr[b];
}
```

**复杂度分析**

希尔排序的时间复杂度受步长的影响，最坏的时间复杂度为`O(n^2)`。



## 5. 归并排序（Merge Sort）

归并排序是采用分治法（Divide and Conquer）的一个典型例子。这个排序的特点是把一个数组打散成小数组，然后再把小数组拼凑再排序，直到最终数组有序。

**步骤**

1.把当前数组分化成n个单位为1的子数组，然后两两比较合并成单位为2的n/2个子数组

2.继续进行这个过程，按照2的倍数进行子数组的比较合并，直到最终数组有序

```java
private void sort(int[] nums) {
    int[] temp = new int[nums.length];
    sort(nums, 0, nums.length - 1, temp);
}

private int[] sort(int[] nums, int low, int high, int[] temp) {
    if (low < high) {
        int mid = (low + high) / 2;
        // 左边排序
        sort(nums, low, mid, temp);
        // 右边排序
        sort(nums, mid + 1, high, temp);
        // 左右归并
        merge(nums, low, mid, high, temp);
    }
    return nums;
}

private void merge(int[] nums, int low, int mid, int high, int[] temp) {
    int i = low;//左序列指针
    int j = mid + 1;//右序列指针
    int t = 0;//临时数组指针
    while (i <= mid && j <= high) {
        if (nums[i] <= nums[j]) {
            temp[t++] = nums[i++];
        } else {
            temp[t++] = nums[j++];
        }
    }
    while (i <= mid) {//将左边剩余元素填充进temp中
        temp[t++] = nums[i++];
    }
    while (j <= high) {//将右序列剩余元素填充进temp中
        temp[t++] = nums[j++];
    }
    t = 0;
    //将temp中的元素全部拷贝到原数组中
    while (low <= high) {
        nums[low++] = temp[t++];
    }
}
```

**复杂度分析**


## 6. 快速排序（Quick Sort）

快速排序也是利用分治法实现的一个排序算法。快速排序和归并排序不同，它不是一半一半的分子数组，而是选择一个基准数，把比这个数小的挪到左边，把比这个数大的移到右边。然后不断对左右两部分也执行相同步骤，直到整个数组有序。

**步骤**

1. 用一个基准数将数组分成两个子数组
2. 将大于基准数的移到右边，小于的移到左边
3. 递归的对子数组重复执行1，2，直到整个数组有序


```java
/**
 * 查找出中轴（默认是最低位low）的在numbers数组排序后所在位置
 * 
 * @param numbers 带查找数组
 * @param low   开始位置
 * @param high  结束位置
 * @return  中轴所在位置
 */
public static int getMiddle(int[] numbers, int low, int high) {
    int temp = numbers[low]; //数组的第一个作为中轴
    while (low < high) {
	    while (low < high && numbers[high] >= temp) {
	        high--;
	    }
	    numbers[low] = numbers[high];//比中轴小的记录移到低端
	    while (low < high && numbers[low] < temp) {
	        low++;
	    }
	    numbers[high] = numbers[low] ; //比中轴大的记录移到高端
    }
    numbers[low] = temp ; //中轴记录到尾
    return low ; // 返回中轴的位置
}

/**
 * 
 * @param numbers 带排序数组
 * @param low  开始位置
 * @param high 结束位置
 */
public static void quickSort(int[] numbers, int low, int high) {
    if (low < high) {
    　　int middle = getMiddle(numbers, low, high); //将numbers数组进行一分为二
    　　quickSort(numbers, low, middle - 1);   //对低字段表进行递归排序
    　　quickSort(numbers, middle + 1, high); //对高字段表进行递归排序
    }
}
```

**复杂度分析**

快速排序是通常被认为在同数量级（O(nlog2n)）的排序方法中平均性能最好的，但若初始序列按关键码有序或基本有序时，快排序反而蜕化为冒泡排序。为改进之，通常以“三者取中法”来选取基准记录，即将排序区间的两个端点与中点三个记录关键码居中的调整为支点记录。快速排序是一个不稳定的排序方法。时间复杂度`O(nlog2n)`，空间复杂度是`O(n)`。

## 7. 堆排序（Heap Sort）

原理：堆排序就是把一个无序数组构建成一个最大堆，然后将堆顶最大数取出，将剩余的堆继续调整为最大堆，再次将堆顶的最大数取出，这个过程持续到剩余数只有一个时结束。

在了解算法之前，首先了解在一维数组中节点的下标：

- i节点的父节点 parent(i) = floor((i-1)/2) 
- i节点的左子节点 left(i) = 2i + 1
- i节点的右子节点 right(i) = 2i + 2

**步骤**

1.构建最大堆：这一步的目的是将一个无序的数组构建成一个满足完全二叉树规则的最大堆，首先从最后一个叶子节点的父节点开始遍历，如果这个父节点比子节点（这里还需要先比较左孩子和右孩子，如果左孩子比右孩子小，那么久比较父节点和右孩子）小，那么就交换，然后将操作索引移动到交换的子节点的左孩子上（如果存在），否则就退出循环，这样一直往前循环到堆顶位置，找出最大堆。

2.交换数组第一个元素和最后一个元素，即移走最大堆堆顶，重新调整最大堆，此时的堆长度要-1，因为经过第一次无序的构建最大堆后，此时的数组除了第一个节点，其余均满足完全二叉树，所以接下来的调整只需要从第一个节点开始比较，将其移动到满足规则的位置。

3.重复执行第二步，直到所有堆顶都已移除。


```java
private void heapSort(int[] nums) {
    int n = nums.length;
    for (int i = n / 2 - 1; i >= 0; i--) {
        buildMaxHeap(nums, i, n);
    }

    for (int i = n - 1; i >= 0; i--) {
        int temp = nums[i];
        nums[i] = nums[0];
        nums[0] = temp;
        buildMaxHeap(nums, 0, i);
    }
}

/**
 * 对数组建最大堆
 */
public void buildMaxHeap(int[] nums, int position, int end) {
    // 当前操作节点
    int current = nums[position];
    // 左孩子索引
    int child = position * 2 + 1;

    while (child < end) {
        if (child + 1 < end && nums[child] < nums[child + 1]) {
            // 如果当前节点存在右子节点且右子节点值比左子节点大，那么将左子节点索引变成右子节点索引
            child++;
        }
        if (current < nums[child]) {
            // 如果当前节点值小于子节点，就交换
            nums[position] = nums[child];
            // 把子节点索引作为当前节点索引，并计算其子节点索引，继续重复上述步骤
            position = child;
            child = position * 2 + 1;
        } else {
            break;
        }
    }
    nums[position] = current;
}
```

**复杂度分析**

堆执行一次调整需要`O(logn)`的时间，在排序过程中需要遍历所有元素执行堆调整，所以最终时间复杂度是`O(nlogn)`。空间复杂度是`O(n)`。

## 参考文章

- [必须知道的八大种排序算法](http://www.jianshu.com/p/8c915179fd02)
- [经典排序算法总结与实现](http://wuchong.me/blog/2014/02/09/algorithm-sort-summary/)
- [图解排序算法(二)之希尔排序](http://www.cnblogs.com/chengxiao/p/6104371.html)
- [图解排序算法(四)之归并排序](http://www.cnblogs.com/chengxiao/p/6194356.html)
- [堆排序C++实现](http://segmentfault.com/a/1190000002466215)
- [常见排序算法 - 堆排序 (Heap Sort)](http://bubkoo.com/2014/01/14/sort-algorithm/heap-sort/)

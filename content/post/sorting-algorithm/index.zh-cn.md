---
title: Sorting Algorithm
author: Jiaqi Gao
description: 
date: 2023-06-03T00:14:15+08:00
lastmod: 2023-06-03T00:14:15+08:00
slug: sorting-algorithm
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - Algorithm
  - DataStructure
tags:
  - Sort
series:
aliases:
---

## 概述
| 排序算法 | 平均时间复杂度 | 最好情况 | 最坏情况 | 空间复杂度 | 排序方式 | 稳定性 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 冒泡排序 | O(n^2) | O(n) | O(n^2) | O(1) | 内部排序 | 稳定 |
| 选择排序 | O(n^2) | O(n^2) | O(n^2) | O(1) | 内部排序 | 不稳定 |
| 插入排序 | O(n^2) | O(n) | O(n^2) | O(1) | 内部排序 | 稳定 |
| 希尔排序 | O(nlogn) | O(nlog^2n) | O(nlog^2n) | O(1) | 内部排序 | 不稳定 |
| 归并排序 | O(nlogn) | O(nlogn) | O(nlogn) | O(n) | 外部排序 | 稳定 |
| 快速排序 | O(nlogn) | O(nlogn) | O(n^2) | O(logn) | 内部排序 | 不稳定 |
| 堆排序 | O(nlogn) | O(nlogn) | O(nlogn) | O(1) | 内部排序 | 不稳定 |
| 计数排序 | O(n+k) | O(n+k) | O(n+k) | O(k) | 外部排序 | 稳定 |
| 桶排序 | O(n+k) | O(n+k) | O(n^2) | O(n+k) | 外部排序 | 稳定 |
| 基数排序 | O(n*k) | O(n*k) | O(n*k) | O(n+k) | 外部排序 | 稳定 |

**稳定性**：在原序列中，`r[i]=r[j]`，且`r[i]`在`r[j]`之前，而在排序后的序列中，`r[i]`仍在`r[j]`之前，则称这种排序算法是稳定的。
**排序方式**：
* **内部排序**：指待排序列完全存放在内存中所进行的排序过程。
* **外部排序**：指待排序记录存储在外存储器上，无法一次全部放入内存，期间需要内存与外存储器进行多次数据交换，最终完成的排序过程。

## 实现
设待排序序列长度为`N`

### 冒泡排序｜Bubble Sort
#### 基础算法
**思想**
> 1. 比较相邻两个元素，如果前面的元素大于后面的元素，就将这两个元素交换。
> 2. 对数组的第`0`个元素到`N-1`个元素进行一次遍历后，最大的一个元素就移动到数组第`N-1`个位置。
> 3. `N=N-1`，如果`N`不为`0`就重复前面二步，否则排序完成。

**实现**
```Java
/**
 * @param arr
 */
public static void bubbleSort(int[] arr) {
    int i, j;
    for (i = 0; i < arr.length; i++) {
        for (j = 1; j < arr.length - i; j++) {
            if (arr[j - 1] > arr[j]) {
                int temp = arr[j - 1];
                arr[j - 1] = arr[j];
                arr[j] = temp;
            }
        }
    }
}
```

#### 优化算法
**思想**
> 基于基础算法，若有一次遍历中未发生数据交换，则排序已经完成。

**实现**
```Java
/**
 * @param arr
 */
public static void bubbleSort(int[] arr) {
    int j, k = arr.length;
    boolean flag = true;
    while (flag) {
        flag = false;
        for (j = 1; j < k; j++) {
            if (arr[j - 1] > arr[j]) {
                int temp = arr[j - 1];
                arr[j - 1] = arr[j];
                arr[j] = temp;
                flag = true;
            }
        }
        k--;
    }
}
```

### 选择排序｜Selection Sort
**思想**
> 1. 对数组进行一次遍历，找到最小元素，并与待排序序列首个元素交换。
> 2. `N=N-1`，若`N>0`，重复第一步，若`N=0`，排序完成。

**实现**
```Java
/**
 * @param arr
 */
public static void selectionSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        int temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
}
```

### 插入排序｜Insertion Sort
**思想**
> 1. 在待排序序列中取出一个元素，遍历已排序序列，将元素插入到合适位置。
> 2. `N=N-1`，若`N>0`，重复第一步，若`N=0`，排序完成。

**实现**
```Java
/**
 * @param arr
 */
public static void insertionSort(int[] arr) {
    int n = arr.length;
    for (int i = 1; i < n; i++) {
        for (int j = i; j > 0 && arr[j] < arr[j - 1]; j--) {
            int temp = arr[j];
            arr[j] = arr[j - 1];
            arr[j - 1] = temp;
        }
    }
}
```

### 希尔排序｜Shell Sort
**思想**
> 希尔排序是把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

**实现**
```Java
/**
 * @param arr
 */
public static void shellSort(int[] arr) {
    int length = arr.length;
    int temp;
    for (int step = length / 2; step >= 1; step /= 2) {
        for (int i = step; i < length; i++) {
            temp = arr[i];
            int j = i - step;
            while (j >= 0 && arr[j] > temp) {
                arr[j + step] = arr[j];
                j -= step;
            }
            arr[j + step] = temp;
        }
    }
}
```

### 归并排序｜Marge Sort
**思想**
> 分治法：
> * 分割：递归地把当前序列平均分割成两半。
> * 集成：在保持元素顺序的同时将上一步得到的子序列集成到一起（归并）。

**实现**
* 递归版
```Java
/**
 * @param arr
 */
static void mergeSortRecursive(int[] arr, int[] result, int start, int end) {
    if (start >= end)
        return;
    int len = end - start, mid = (len >> 1) + start;
    int start1 = start, end1 = mid;
    int start2 = mid + 1, end2 = end;
    mergeSortRecursive(arr, result, start1, end1);
    mergeSortRecursive(arr, result, start2, end2);
    int k = start;
    while (start1 <= end1 && start2 <= end2)
        result[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
    while (start1 <= end1)
        result[k++] = arr[start1++];
    while (start2 <= end2)
        result[k++] = arr[start2++];
    for (k = start; k <= end; k++)
        arr[k] = result[k];
}
public static void mergeSort(int[] arr) {
    int len = arr.length;
    int[] result = new int[len];
    mergeSortRecursive(arr, result, 0, len - 1);
}
```

* 迭代版
```Java
public static void mergeSort(int[] arr) {
    int[] orderedArr = new int[arr.length];
    for (int i = 2; i < arr.length * 2; i *= 2) {
        for (int j = 0; j < (arr.length + i - 1) / i; j++) {
            int left = i * j;
            int mid = left + i / 2 >= arr.length ? (arr.length - 1) : (left + i / 2);
            int right = i * (j + 1) - 1 >= arr.length ? (arr.length - 1) : (i * (j + 1) - 1);
            int start = left, l = left, m = mid;
            while (l < mid && m <= right) {
                if (arr[l] < arr[m]) {
                    orderedArr[start++] = arr[l++];
                } else {
                    orderedArr[start++] = arr[m++];
                }
            }
            while (l < mid)
                orderedArr[start++] = arr[l++];
            while (m <= right)
                orderedArr[start++] = arr[m++];
            System.arraycopy(orderedArr, left, arr, left, right - left + 1);
        }
    }
}
```

### 快速排序｜Quick Sort
**思想**
> 分治法：
> 1. 挑选基准值：从数列中挑出一个元素，称为“基准”。
> 2. 分割：重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（与基准值相等的数可以到任何一边）。
> 3. 递归排序子序列：递归地将小于基准值元素的子序列和大于基准值元素的子序列排序。

**实现**
```Java
/**
 * @param arr
 * @param low
 * @param high
 */
static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int pointer = low;
    for (int i = low; i < high; i++) {
        if (arr[i] <= pivot) {
            int temp = arr[i];
            arr[i] = arr[pointer];
            array[pointer] = temp;
            pointer++;
        }
    }
    int temp = arr[pointer];
    arr[pointer] = arr[high];
    arr[high] = temp;
    return pointer;
}

public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int position = partition(arr, low, high);
        quickSort(arr, low, position - 1);
        quickSort(arr, position + 1, high);
    }
}
```

### 堆排序｜Heap Sort
**思想**
> 1. 建立一个堆`H[0...n-1]`。
> 2. 把堆首（最大值）和堆尾互换。
> 3. 把堆的尺寸缩小`1`，把新的数组顶端数据调整到相应位置。
> 4. 重复步骤`2`，直到堆的尺寸为`1`。

**实现**
```Java
/**
 * @param arr
 */
public void heapSort(int[] arr) {
    int len = arr.length - 1;
    int beginIndex = (arr.length >> 1) - 1;
    for (int i = beginIndex; i >= 0; i--)
        maxHeapify(i, len);
    for (int i = len; i > 0; i--) {
        swap(0, i);
        maxHeapify(0, i - 1);
    }
}
private void swap(int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
private void maxHeapify(int index, int len) {
    int li = (index << 1) + 1;
    int ri = li + 1;
    int cMax = li;
    if (li > len) return;
    if (ri <= len && arr[ri] > arr[li])
        cMax = ri;
    if (arr[cMax] > arr[index]) {
        swap(cMax, index);
        maxHeapify(cMax, len);
    }
}
```

### 计数排序｜Count Sort
**思想**
> 1. 找出待排序数组中最大和最小的元素。
> 2. 统计数组中每个值为`i` 的元素出现的次数，存入计数数组`C`的第`i-minValue`项。
> 3. 对所有的计数累加。
> 4. 反向填充数组：将每个元素`i`放在新数组的第`C[i]`项，每放一个元素就将`C[i]`减去`1`。

**实现**
```Java
/**
 * @param arr
 * @return b
 */
public static int[] countSort(int[] arr){
    int b[] = new int[arr.length];
    int max = arr[0], min = arr[0];
    for(int i : arr){
        if(i > max){
            max = i;
        }
        if(i < min){
            min = i;
        }
    }
    int k = max - min + 1;
    int c[] = new int[k];
    for(int i = 0; i < arr.length; ++i){
        c[a[i] - min] += 1;
    }
    for(int i = 1; i < c.length; ++i){
        c[i] = c[i] + c[i - 1];
    }
    for(int i = arr.length - 1; i >= 0; --i){
        b[--c[arr[i] - min]] = arr[i];
    }
    return b;
}
```

### 桶排序｜Bucket Sort
**思想**
> 1. 设置一个定量的数组当作空桶。
> 2. 遍历序列，并把元素放到对应的桶去。
> 3. 对每一个非空桶进行排序。
> 4. 再将非空桶中元素放回原队列。

**实现**
```Java
/**
 * @param arr
 */
public void bucketSort(int[] arr) {
    int max = arr[0], min = arr[0];
    for (int a : arr) {
        if (max < a)
            max = a;
        if (min > a)
            min = a;
    }
    int bucketNum = max / 10 - min / 10 + 1;
    List buckList = new ArrayList<List<Integer>>();
    for (int i = 1; i <= bucketNum; i++) {
        buckList.add(new ArrayList<Integer>());
    }
    for (int i = 0; i < arr.length; i++) {
        int index = indexFor(arr[i], min, 10);
        ((ArrayList<Integer>) buckList.get(index)).add(arr[i]);
    }
    ArrayList<Integer> bucket = null;
    int index = 0;
    for (int i = 0; i < bucketNum; i++) {
        bucket = (ArrayList<Integer>) buckList.get(i);
        insertSort(bucket);
        for (int k : bucket) {
            arr[index++] = k;
        }
    }
}

private int indexFor(int a, int min, int step) {
    return (a - min) / step;
}

private void insertSort(List<Integer> bucket) {
    for (int i = 1; i < bucket.size(); i++) {
        int temp = bucket.get(i);
        int j = i - 1;
        for (; j >= 0 && bucket.get(j) > temp; j--) {
            bucket.set(j + 1, bucket.get(j));
        }
        bucket.set(j + 1, temp);
    }
}
```

### 基数排序｜Radix Sort
**思想**
> 1. 将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。
> 2. 从最低位开始，依次进行一次排序。
> 3. 排序完成以后，数列就变成一个有序序列。

**实现**
```Java
/**
 * @param arr
 */
public static void radixSort(int[] arr) {
    if (null == arr || 0 == arr.length) return;
    int k = maxbit(arr);
    int radix = 1;
    for (int i = 0; i < k; i++) {
        countingSort(arr, radix);
        radix *= 10;
    }
}

private static int maxbit(int[] arr) {
    int max = arr[0];
    for (int i = 1; i < arr.length; i++) max = max < arr[i] ? arr[i] : max;
    int d = 1;
    while ((max = max / 10) > 0) d++;
    return d;
}

public static void countingSort(int[] arr, int radix) {
    int[] counts = new int[10];
    int len = arr.length;
    int[] buffer = new int[len];
    for (int i = 0; i < len; i++) counts[(arr[i] / radix) % 10]++;
    for (int i = 1; i < counts.length; i++) counts[i] = counts[i] + counts[i - 1];
    for (int i = len - 1; i >= 0; i--) {
        buffer[counts[(arr[i] / radix ) % 10] - 1] = arr[i];
        counts[(arr[i] / radix) % 10] = counts[(arr[i] / radix) % 10] - 1;
    }
    for (int i = 0; i < len; i++) arr[i] = buffer[i];
}
```
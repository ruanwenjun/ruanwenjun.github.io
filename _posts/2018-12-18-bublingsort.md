---
layout: post
title: 冒泡排序、选择排序、插入排序
tags: [数据结构与算法]
---
目录：
- [冒泡排序](#冒泡排序)
- [选择排序](#选择排序)
- [插入排序](#插入排序)

## 冒泡排序

冒泡排序就是指每次比较相邻的两个数，然后让大的那个数在右边，这样一直下去，就可以将当前最大的数放到最后面。这就有点类似水里面的鱼吐泡泡，泡泡越往上走就变得越大。



那么为什么不直接从头到尾比较找到最大的数放到后面，而是基于一种两两比较交换的思路呢？我想原因应该是为了解决初始有序的情况，当初始有序的时候就不需要进行交换，那么就可以引入一个标示，当一次都没有交换的时候，就可以不用再进行后面的排序了

代码：
```java

public class Test {
    public static void main(String[] args) {
        int[] array = generateArray();
        System.out.println(Arrays.toString(array));
        bubblingSort(array);
    }

    /**
     * 冒泡排序：每次往后比较，将最大的泡放到后面
     * @param array
     */
    public static void bubblingSort(int[] array) {
        int len = array.length - 1;
        boolean flag = true; // 当前是否发生交换的标示，如果没有发生交换，则代表已经是有序的，仔细想想是不是这么个道理
        for (int i = 0; i < len; i++) {
            // 找到最大的泡
            for (int j = 0; j < len - i; j++) {
                // 比较并交换
                if (array[j] > array[j + 1]) {
                    int index = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = index;
                }
            }
            System.out.println(Arrays.toString(array));
        }
    }

    /**
     * 生成数组
     * @return
     */
    public static int[] generateArray() {
        Random random = new Random();
        int[] array = new int[random.nextInt(20)];
        for (int i = 0; i < array.length; i++) {
            array[i] = random.nextInt(20);
        }
        return array;
    }
}
```

## 选择排序

选择排序是找到当前最小的数，加到已经排好序的最后。其实感觉跟冒泡差不多，冒泡可以看成是找到当前最大的泡，加到已知泡到最左边。区别就是冒泡是两两交换，选择排序是找最小的，不进行交换。所以冒泡排序当初始有序的时候就只需要遍历一遍就可以了，但是选择排序不管是不是有序都要遍历N遍

代码：
```java
public class Test {
    public static void main(String[] args) {
        int[] array = generateArray();
        System.out.println(Arrays.toString(array));
        selectSort(array);
    }

    /**
     * 选择排序：每次选择最小的数，加到有序数组的最后面
     *
     * @param array
     */
    public static void selectSort(int[] array) {
        for (int i = 0; i < array.length; i++) {  // 当前有序数组的大小
            int min = i; // 记录当前最小值的下标
            for (int j = i; j < array.length; j++) {  // 遍历剩下的无序数组找到最小值
                if (array[j] < array[min]) {
                    min = j;
                }
            }
            int index = array[min];
            array[min] = array[i];
            array[i] = index;
            System.out.println(Arrays.toString(array));
        }
    }

    /**
     * 生成数组
     *
     * @return
     */
    public static int[] generateArray() {
        Random random = new Random();
        int[] array = new int[random.nextInt(20)];
        for (int i = 0; i < array.length; i++) {
            array[i] = random.nextInt(20);
        }
        return array;
    }
}
```

## 插入排序

遍历数组，将数组里的每一个元素插入到前面已经排好序的有序数组中，这点跟选择排序有点像，都是将数加到有序数组中，但是插入排序是插的时候找位置插，选择排序是直接插到最后面了。
代码：
```java

public class Test {
    public static void main(String[] args) {
        int[] array = generateArray();
        System.out.println(Arrays.toString(array));
        insertSort(array);
    }

    /**
     * 插入排序：每次将数插入到已知有序数组里面
     *
     * @param array
     */
    public static void insertSort(int[] array) {
        for (int i = 1; i < array.length; i++) {
            int index = array[i]; // 将每一个元素插入到已知有序数组中
            int j = i - 1;
            for (; j >= 0 && array[j] > index; j--) {
                array[j + 1] = array[j];   // 往右移
            }
            // 插入到当前位置
            array[j + 1] = index;
            System.out.println(Arrays.toString(array));
        }
    }

    /**
     * 生成数组
     *
     * @return
     */
    public static int[] generateArray() {
        Random random = new Random();
        int[] array = new int[random.nextInt(20)];
        for (int i = 0; i < array.length; i++) {
            array[i] = random.nextInt(20);
        }
        return array;
    }
}
```

## 比较

|算法|平均时间复杂度|最好时间复杂度|最坏时间复杂度|稳定性|
| ------ | ------ | ------ |------ | ------ |
|冒泡排序|O(n^2)|O(n)|O(n^2)|稳定|
|选择排序|O(n^2)|O(n^2)|O(n^2)|不稳定|
|插入排序|O(n^2)|O(n)|O(n^2)|稳定|

关于稳定性，只有选择排序是不稳定的。这里给个例子：比如一个数组 [4,4,2,1] ，
那么选择排序的过程是
[1,4,2,4]   // 两个4的顺序变了
[1,2,4,4]   




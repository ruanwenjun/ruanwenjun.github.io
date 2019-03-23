---
layout: post
title: topK问题
tags: [算法与数据结构]
---

- [无序数组中找topK问题](#无序数组中找topK问题)
- [有序数组中topK问题](#有序数组中topK问题)
- [数组中找中位数问题](#数组中找中位数问题)

# 无序数组中找topK问题
例如给一个无序数组，需要找到其中前100大的数。
思路：
1. 可以建立一个包含100个元素的最小堆，然后将数组中的元素依次加入到最小堆中，那么最后堆中的100个元素就是top100大，堆顶就是第top100大,时间复杂度O(nlog100),空间复杂度O(1)
2. 可以采用快排partition的思路，如果当前分割点右边刚好有99个元素，那么当前分割点就是第top100大。时间复杂度O(n)。
# 有序数组中topK问题
如果给一有序数组，需要找其中前K大的数
思路：
1. 数组是有序的，那么就可以直接根据下标找到后面K个元素，时间复杂度O(1)
如果给定两个有序数组，需要找到其中前K大的数
思路：
1. 维护双指针，遍历,时间复杂度O(K)
2. 每次将k减少k/2,让k逐步逼近1，时间复杂度O(logK)
# 数组中找中位数问题
给定一无序数组，需要找到其中的中位数
思路：
1. 可以把问题转换为在无序数组中找到第topK问题，可以采用partition思路
如果给定两个有序数组，需要找到其中的中位数
思路：
1. 同样可以把问题转换为在两个有序数组找topK问题，但是这里使用双指针的话时间复杂度会是O((M + N)/2)），可以采用二分思路逐步逼近，把时间复杂度缩至O(log(M+N).

代码：
```java
class Solution {
    // 寻找两个有序数组的中位数
    // 问题转换为求第K小的数
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if(nums1 == null || nums1.length == 0){
            // 求nums2的中位数
            return nums2.length % 2 == 0 
            ? (nums2[nums2.length / 2] + nums2[nums2.length / 2 - 1]) / 2.0 
            : nums2[nums2.length / 2];
        }
        if(nums2 == null || nums2.length == 0){
            return nums1.length % 2 == 0 
            ? (nums1[nums1.length / 2] + nums1[nums1.length / 2 - 1]) / 2.0 
            : nums1[nums1.length / 2];
        }
        int len = nums1.length + nums2.length;
        return len % 2 == 0 
        ? (topK(nums1,nums2,0,0,len/2)+topK(nums1,nums2,0,0,len/2+1))/2.0 
        : topK(nums1,nums2,0,0,len/2 + 1);
    }

    // 找两个有序数组的第k小的数
    // 1 3 5 7 9
    // 2 4 6 8 10
    // 思路：逐步减小k，并且每次减少的幅度k / 2
    public int topK(int[] nums1,int[] nums2,int start1,int start2,int k){
        if(start1 >= nums1.length){
            return nums2[start2 + k - 1]; // 这里注意需要下标减1
        }
        if(start2 >= nums2.length){
            return nums1[start1 + k - 1];
        }

        if(k == 1){ // 转换成了两个有序数组中找第一小的数
            return Math.min(nums1[start1] , nums2[start2]);
        }

        if(start1 + k / 2 > nums1.length){ // 肯定不会在nums2的前 k / 2
            return topK(nums1,nums2,start1,start2 + k / 2,k - k / 2);
        }else if(start2 + k / 2 > nums2.length){
            return topK(nums1,nums2,start1 + k / 2,start2,k - k / 2);
        }

        int mid1 = nums1[start1 + k / 2 - 1];
        int mid2 = nums2[start2 + k / 2 - 1];
        if(mid1 > mid2){ // 移除nums2的前 k / 2
            return topK(nums1,nums2,start1,start2 + k / 2,k - k / 2);
        }else {
            return topK(nums1,nums2,start1 + k / 2,start2,k - k/2);
        }
    }
}

```
---
title: leetcode
date: 2026-01-31 21:57:17
tags: 学习
categories: 笔记
---

## leetcode刷题记录

### 35.二分查找

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
请必须使用时间复杂度为 O(log n) 的算法。

示例 1:
  >输入: nums = [1,3,5,6], target = 5
  >输出: 2

示例 2:
  >输入: nums = [1,3,5,6], target = 2
  >输出: 1

示例 3:
  >输入: nums = [1,3,5,6], target = 7
  >输出: 4

对于上述问题，可以使用二分查找算法来实现，代码如下：

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0; 
        int right = nums.size() - 1;
        
        while(left <= right){
            int mid = left + (right - left) / 2;

            if(nums[mid] == target){
                return mid;
            }else if(nums[mid] < target){
                left = mid + 1;
            }else{
                right = mid - 1;
            }
        }
        return left;
    }
};
```

如果说没有“返回它将会被按顺序插入的位置”这个要求的话，只需要将最后return变成-1即可，即找到了返回mid,没找到就返回-1。

### 26.双指针去重

给你一个 非严格递增排列 的数组 nums，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。然后返回 nums 中唯一元素的个数。

考虑 nums 的唯一元素的数量为 k。去重后，返回唯一元素的数量 k。

nums 的前 k 个元素应包含 排序后 的唯一数字。下标 k - 1 之后的剩余元素可以忽略。

示例 1：
  >输入：nums = [1,1,2]
  >输出：2, nums = [1,2,_]
  >解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。

示例 2：
  >输入：nums = [0,0,1,1,1,2,2,3,3,4]
  >输出：5, nums = [0,1,2,3,4,_,_,_,_,_]
  >解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。

对于上述问题，可以使用双指针算法来实现，代码如下：

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.empty()) return 0;

        int left = 0;
        for(int right = 1; right < nums.size(); right++){
            if(nums[left] != nums[right]){
                left++;
                nums[left] = nums[right];
            }
        }
        return left + 1;
    }
};
```

---
layout: article
title:  在排序数组中查找元素的第一个和最后一个位置
tags: algorithm binary_search 
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-31-algorithm-searchRange
comment: true
mermaid: true
chart: true
---

## 题目描述

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。如果数组中不存在目标值 target，返回 [-1, -1]。   
进阶：  
你可以设计并实现时间复杂度为 O(log n) 的算法解决此问题吗？

示例 1：  
输入：nums = [5,7,7,8,8,10], target = 8  
输出：[3,4]  

示例 2：  
输入：nums = [5,7,7,8,8,10], target = 6  
输出：[-1,-1]  

示例 3：  
输入：nums = [], target = 0  
输出：[-1,-1]   

提示：  

- $0 <= nums.length <= 10^5$  
- $-10^9 <= nums[i] <= 10^9$  
- nums 是一个非递减数组  
- $-10^9 <= target <= 10^9$  


## 解题思路

### 二分搜索

这道题最直观的思路肯定是从前到后遍历一遍，用两个变量记录第一次和最后一次遇见target的下标，但是这个方法的时间复杂度为O(n)，而且没利用到数组是升序排列的条件。  
由于数组已经排序，因此整个数组是单调递增的，我们可以利用二分查找加速查找过程。  
考虑targer的开始和结束位置，等价于找到第一个等于target的位置和第一个大于target的位置减1。因为target可能不存在于数组中，所以我们需要检验找到的下表是否符合条件。

## 代码

golang实现：

```go
func searchRange(nums []int, target int) []int {
	if len(nums) <= 0 {
		return []int{-1, -1}
	}
	low := lowHelper(nums, target)
	high := highHelper(nums, target) - 1
	if low == len(nums) || nums[low] != target { // target有可能不在nums中
		return []int{-1, -1}
	}
	return []int{low, high}
}

func highHelper(nums []int, target int) int {
	l, r := 0, len(nums)
	for l < r {
		mid := l + (r-l)/2
		if nums[mid] > target {
			r = mid
		} else {
			l = mid + 1
		}
	}
	return l
}

func lowHelper(nums []int, target int) int {
	l, r := 0, len(nums)
	for l < r {
		mid := l + (r-l)/2
		if nums[mid] >= target {
			r = mid
		} else {
			l = mid + 1
		}
	}
	return l
}
```

## 时间和空间复杂度

- 时间复杂度：O(logn) ，其中n为数组的长度。二分查找的时间复杂度为 O(logn)，一共会执行两次，因此总时间复杂度为 O(logn)。
- 空间复杂度：O(1) 。只需要常数空间存放若干变量。

---
layout: article
title: 四数之和
tags: algorithm
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-08-algorithm-fourSum
comment: true
mermaid: true
chart: true
---

# 题目描述

给你一个由 n 个整数组成的数组 nums ，和一个目标值 target 。请你找出并返回满足下述全部条件且不重复的四元组 [nums[a],nums[b], nums[c], nums[d]] （若两个四元组元素一一对应，则认为两个四元组重复）：

0 <= a, b, c, d < n
a、b、c 和 d 互不相同
nums[a] + nums[b] + nums[c] + nums[d] == target

你可以按 任意顺序 返回答案 。

示例 1：

输入：nums = [1,0,-1,0,-2,2], target = 0
输出：[[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]]

示例 2：
输入：nums = [2,2,2,2,2], target = 8
输出：[[[2,2,2,2]]]

提示：

1 <= nums.length <= 200
-10^9 <= nums[i] <= 10^9
-10^9 <= target <= 10^9

# 解题思路

对于这个题目，容易想到暴力解决四重循环直接遍历出答案，再用hash表去重，这是一种解法但显然不是比较好的解法。显然该暴力解法时间复杂度为$(n^4)$，我们可以使用排序加双指针的方法把时间复杂度降到$(n^3)$。

为了避免枚举到重复四元组，我们需要保证每一重循环枚举到的元素不少于上一次枚举的元素，且每一层枚举的元素不相同。
为了实现上述要求，我们可以先将输入数组排序，并在循环过程中满足一下两点：
1. 每一次循环枚举的下标需大于上一层循环；
2. 若当前循环中当前元素和前一个元素相同，需跳过该元素。

使用上述方法，可以避免枚举到重复的四元组，但是四层循环还是存在。因为数组已经排序，我们可以使用双指针的方法将后两重循环缩减到一重。假设前两成循环的下标分别为$i$，$j$，其中$i<j$。第三层循环初始化时，left指针指向$j+1$，right指向$n-1$。每次循环计算四个元素的和：
1. 如果它们的和等于target，将该四元组加到结果中；
2. 如果它们的和大于target，right减1；
3. 如果它们的和小于target，left加1。

通过双指针，将剩下两个数的枚举复杂度缩减到$O(n)$了，由于排序的最优时间复杂度可以去到$O(nlogn)$，因此我们的最终时间复杂度为$O(n^3)$。

同时，因为数组是**排序**过的，我们还可以使用剪枝的手法在某些情况下提前退出循环提高效率：
1. 当$i$确定后，如果$nums[i]+nums[i+1]+nums[i+2]+nums[i+3]>target$，显然后面再遍历也不会有小于或等于target的四元组；
2. 当$i$确定后，如果$nums[i]+nums[n-1]+nums[n-2]+nums[n-3]<target$，显然后面再遍历也不会有大于或等于target的四元组；
3. 当$i$和$j$确定后，如果$nums[i]+nums[j]+nums[j+1]+nums[j+2]>target$，显然后面再遍历也不会有小于或等于target的四元组；
4. 当$i$和$j$确定后，如果$nums[i]+nums[j]+nums[n-1]+nums[n-2]<target$，显然后面再遍历也不会有大于或等于target的四元组；

# 代码

golang实现：
```go
import (
	"sort"
)

func fourSum(nums []int, target int) (res [][]int) {
	sort.Ints(nums)
	n := len(nums)
	for i := 0; i < n-3 && nums[i]+nums[i+1]+nums[i+2]+nums[i+3] <= target; i++ { // 跳过相同元素 && 剪枝
		if i > 0 && nums[i] == nums[i-1] || nums[i]+nums[n-1]+nums[n-2]+nums[n-3] < target { // 跳过相同元素 && 剪枝
			continue
		}
		for j := i + 1; j < n-2 && nums[i]+nums[j]+nums[j+1]+nums[j+2] <= target; j++ { // && 剪枝
			if j > i+1 && nums[j] == nums[j-1] || nums[i]+nums[j]+nums[n-1]+nums[n-2] < target { // 跳过相同元素 && 剪枝
				continue
			}
			for left, right := j+1, n-1; left < right; {
				if nums[i]+nums[j]+nums[left]+nums[right] == target {
					res = append(res, []int{nums[i], nums[j], nums[left], nums[right]})
					for left++; left < right && nums[left] == nums[left-1]; left++ { // 跳过相同元素
					}
					for right--; left < right && nums[right] == nums[right+1]; right-- { // 跳过相同元素
					}
				} else if nums[i]+nums[j]+nums[left]+nums[right] > target {
					right--
				} else {
					left++
				}
			}
		}
	}
	return res
}
```

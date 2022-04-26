---
layout: article
title: 非递减数列
tags: algorithm greedy 
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-26-algorithm-checkPossibility
comment: true
mermaid: true
chart: true
---

# 题目描述

给你一个长度为 n 的整数数组 nums ，请你判断在 最多 改变 1 个元素的情况下，该数组能否变成一个非递减数列。  
我们是这样定义一个非递减数列的： 对于数组中任意的 i (0 <= i <= n-2)，总满足 nums[i] <= nums[i + 1]。  

示例 1:  

输入: nums = [4,2,3]  
输出: true  
解释: 你可以通过把第一个 4 变成 1 来使得它成为一个非递减数列。  
 
示例 2:  

输入: nums = [4,2,1]  
输出: false  
解释: 你不能在只改变一个元素的情况下将其变为非递减数列。  
 
提示：  
$n == nums.length$  
$1 <= n <= 10^4$  
$-10^5 <= nums[i] <= 10^5$  

# 解题思路

## 贪心解法

这道题需要思考我们的贪心策略在各种情况下，是否仍是最优解。  
首先思考以下问题：
> 要使数组nums变成一个非递减数列，数组中`最多`有多少个i满足$nums[i]>nums[i+1]$？

假设有两个不同的下标i，j满足上述不等式，不妨设$i<j$:
- 若$i+1<j$，则无论怎么修改$nums[i]$或$nums[i+1]$，都不会影响$nums[j]$和$nums[j+1]$之前都大小关系；修改$nums[j]$或$nums[j+1]$同理，因为，在这种情况下我们无法将nums变成非递减队列。
- 若$i+1=j$，则有$nums[i]>nums[i+1]>nums[i+2]$。同样地，无论怎么修改其中一个数，都无法使这三个数变为$nums[i]<=nums[i+1]<=nums[i+2]$的大小关系。

因此，上面问题的结论是：
> 最多一个

满足这个条件就足够了吗？并不是的，对于满足条件的数组[3,4,1,2]而言，无论怎么修改也无法将其变为非递减数列。因为，若找到了一个满足$nums[i]>nums[i+1]$的i，在修改$nums[i]$或$nums[i+1]$之后，我们还需要检查nums是否变成了非递减数列。  
我们可以将$nums[i]$修改为小于或等于$nums[i+1]$的数，但是由于还需要保证$nums[i]$不少于它之前的数，$nums[i]$应该修改成越大的数越好，所以将$nums[i]$修改成$nums[i+1]$是最好的；同理，对于$nums[i+1]$修改成$nums[i]$是最好的。  

在实际代码中，我们可以做一些优化：
> 修改$nums[i]$为$nums[i+1]$后，还需要保证$nums[i-1]<=nums[i]$仍然成立，即$nums[i-1]<=nums[i+1]$，若该不等式不成立则整个数组必然不是非递减的，则需要修改$nums[i+1]$为$nums[i]$。修改完后判断后续数字的大小关系，在遍历中统计$nums[i]>nums[i+1]$的次数，若超过一次可以直接返回false。



# 代码

golang实现：
```go
func checkPossibility(nums []int) bool {
	count, n := 0, len(nums)-1
	for i := 0; i < n; i++ {
		x, y := nums[i], nums[i+1]
		if x > y {
			count++
			if count > 1 {
				return false
			}
			if i > 0 && y < nums[i-1] {
				nums[i+1] = x
			}
		}
	}
	return true
}
```

# 时间和空间复杂度

- 时间复杂度：$O(n)$，其中n是数组nums的长度。
- 空间复杂度：$O(1)$。

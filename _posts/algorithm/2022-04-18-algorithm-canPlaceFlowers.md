---
layout: article
title: 种花问题
tags: algorithm greedy interval
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-18-algorithm-canPlaceFlowers
comment: true
mermaid: true
chart: true
---

# 题目描述

假设有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花不能种植在相邻的地块上，它们会争夺水源，两者都会死去。  
给你一个整数数组flowerbed表示花坛，由若干 0 和 1 组成，其中 0 表示没种植花，1 表示种植了花。另有一个数n，能否在不打破种植规则的情况下种入 n 朵花？能则返回 true ，不能则返回 false。  

示例 1：  
输入：flowerbed = [1,0,0,0,1], n = 1  
输出：true  

示例 2：  
输入：flowerbed = [1,0,0,0,1], n = 2  
输出：false  

提示：  
flowerbed[i] 为 0 或 1  
flowerbed 中不存在相邻的两朵花  

# 解题思路

## 贪心解法

首选我们明确知道可以种花的两个条件：
1. 当前位置为0；
2. 相邻位置不为1。

在这个题目的贪心策略中，我们只需要遍历一次数组，只考虑数组中当前位置是否可以种花（边界条件特殊处理），因为种上花的地方相邻位置都不能为1，因此：
1. 如果能种花，就种上花，跳到下下个位置；
2. 如果不能种花，直接跳到下下个位置；
 
所以，我们在遍历花坛数组时，对于每一个位置，我们在判断这个位置是否有花：
1. 如果当前位置有花，则跳到下下个位置，i=i+2;
2. 如果当前位置没有花，有几种情况
   1. 如果当前位置的下一个位置有花，则当前不能种花，跳到下一个位置，i=i+1;
   2. 如果当前位置的下一个位置没花，则当前位置可以种花，种上花后(n--)跳到下下个位置，(n--);
   3. 特殊边界情况：当前位置为花坛最后一个位置，且当前位置无花，则当前位置可以种花(n--)。

### 为什么我们在代码中只判断右边界不判断左边界呢？

因为无论是当前位置有花，还是当前位置无花但是种上花了，我们跳的花坛位都是：i=i+2。这让我们当前遍历到的花坛位是下一个可能种花的位置，而不会遍历到出现左边有花需要判断的位置($i=i+2$跳过了)，当然如果遍历到花坛最后一个位置数需要特殊判断的。  
另外我们代码中对于当前位置无花，但是下一个位置有花的情况，会直接走i=i+1这个路径，所以代码中也是包含了这个情况的。

在代码中我们可以剪枝提前退出循环提高效率：当n缩减到0时证明给出的种花要求可以满足，直接提前返回true。

# 代码

golang实现：
```go
func canPlaceFlowers(flowerbed []int, n int) bool {
	if n <= 0 {
		return true
	}
	m := len(flowerbed)
	for i := 0; i < m; i++ {
		if flowerbed[i] == 0 && (i+1 == m || flowerbed[i+1] == 0) {
			n--
			if n == 0 {
				return true
			}
			i++
		} else if flowerbed[i] == 1 {
			i++
		}
	}
	return false
}
```

# 时间和空间复杂度

- 时间复杂度：$O(n)$，其中$n$是区间的数量。我们需要$O(n)$的时间进行遍历。
- 空间复杂度：$O(1)$，需要常数空间保存相关变量。

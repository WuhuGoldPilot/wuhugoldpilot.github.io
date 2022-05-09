---
layout: article
title: 平方数之和
tags: algorithm double_pointer two_sum
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-09-algorithm-judgeSquareSum
comment: true
mermaid: true
chart: true
---

# 题目描述

给定一个非负整数 c ，你要判断是否存在两个整数 a 和 b，使得 $a^2 + b^2 = c$ 。   

示例 1：  
输入：c = 5  
输出：true  
解释：1 * 1 + 2 * 2 = 5  

示例 2：  
输入：c = 3  
输出：false  

提示：  
$0 <= c <= 2^{31} - 1$  

# 解题思路

## 双指针

这是两数之和的题型的变形题，我们可以用双指针的方法解出来。
使用left，right两个指针初始化为0和$\sqrt c$，计算它们的平方和sum：
1. 如果sum=c，说明存在两个数的平方和等于c，直接返回true；
2. 如果sum>c，说明两个指针指向的数字偏大，缩小right指针，righr--；
3. 如果sum<c，说明两个指针指向的数字偏小，扩大left指针，left++；
4. 如果两个指针相遇都没找到符合要求的平方和，证明不存在这样的整数a和b，返回false。

# 代码

golang实现：
```go
import "math"

func judgeSquareSum(c int) bool {
	left, right := 0, int(math.Sqrt(float64(c)))
	for left <= right {
		sum := left*left + right*right
		if sum == c {
			return true
		} else if sum > c {
			right--
		} else {
			left++
		}
	}
	return false
}
```

# 时间和空间复杂度

- 时间复杂度：O($\sqrt c$)。最坏情况下a和b一共枚举了0到$\sqrt c$里的所有整数。
- 空间复杂度：O(1)。


---
layout: article
title: 验证回文字符串 Ⅱ
tags: algorithm double_pointer 
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-11-algorithm-validPalindrome
comment: true
mermaid: true
chart: true
---

## 题目描述

给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。  

示例 1:  
输入: s = "aba"  
输出: true  

示例 2:  
输入: s = "abca"  
输出: true  
解释: 你可以删除c字符。  

示例 3:  
输入: s = "abc"  
输出: false  

提示:  
$1 <= s.length <= 10^5$  
s 由小写英文字母组成  

## 解题思路

### 双指针

首先思考如何判断一个字符串是回文串？我们可以使用双指针的方法，左右指针初始化为字符串的开头和结尾，通过不断移动这两个指针比较它们指向的字符是否相等，不相等就移动左右指针，进行下一轮的比较，直至左右指针相遇它们经过的字符都相等，那么这个字符串就是回文串。  
题目中要求最多可以删除一个字符，验证是否可以成为回文串也可以使用上面的方法。我们在比较左右指针指向的元素时，如果遇到一个不相等的字符，那么有两种删除字符的选择：一种是删除左指针指向的字符；一种是删除右指针指向的字符。将这两种情况我们都需要验证删除了后是否符合回文串的定义，如果其中一种情况符合就证明该字符串可以删除最多一个字符成为回文串。验证是回文串的方法在上面已经说了，因此我们可以很自然地写出这个算法的答案。

## 代码

golang实现：

```go
func validPalindrome(s string) bool {
	left, right := 0, len(s)-1
	for left < right {
		if s[left] == s[right] {
			left++
			right--
		} else {
			flag1, flag2 := true, true
			for i, j := left+1, right; i < j; i, j = i+1, j-1 {
				if s[i] != s[j] {
					flag1 = false
					break
				}
			}
			for i, j := left, right-1; i < j; i, j = i+1, j-1 {
				if s[i] != s[j] {
					flag2 = false
					break
				}
			}
			return flag1 || flag2
		}
	}
	return true
}
```

## 时间和空间复杂度

- 时间复杂度：O(n)。其中n是字符串的长度。判断整个字符串是否是回文字符串的时间复杂度是 O(n)，遇到不同字符时，判断两个子串是否是回文字符串的时间复杂度也都是 O(n)。
- 空间复杂度：O(1)。

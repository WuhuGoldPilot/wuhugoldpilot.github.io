---
layout: article
title: 通过删除字母匹配到字典里最长单词
tags: algorithm double_pointer 
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-16-algorithm-findLongestWord
comment: true
mermaid: true
chart: true
---

## 题目描述

给你一个字符串 s 和一个字符串数组 dictionary ，找出并返回 dictionary 中最长的字符串，该字符串可以通过删除 s 中的某些字符得到。如果答案不止一个，返回长度最长且字母序最小的字符串。如果答案不存在，则返回空字符串。  

示例 1：  
输入：s = "abpcplea", dictionary = ["ale","apple","monkey","plea"]  
输出："apple"  

示例 2：  
输入：s = "abpcplea", dictionary = ["a","b","c"]  
输出："a"  

提示：  
1 <= s.length <= 1000  
1 <= dictionary.length <= 1000  
1 <= dictionary[i].length <= 1000  
s 和 dictionary[i] 仅由小写英文字母组成   

## 解题思路

### 双指针

根据题意，我们需要解决两个问题：
1. 如何判断dictionary中的字符串t是否可以通过删除s中的某些字符得到；
2. 如何找到长度最长且字典序最小的字符串。

第一个问题实际上就是判断t是否是s的子序列，因此我们只要找到任意一种t在s中出现的方式，即可认为t是s的子序列。而当我们从前往后匹配时，可以发现每次贪心地匹配最靠前的字符是符合最有决策的。
> 假定当钱需要匹配字符c，而字符c在s中的位置$x_1$和$x_2$出现（$x_1<x_2>$），那么贪心取$x_1$是最优解，因为$x_2$后面能取到的字符$x_1$也能取到，并且通过$x_1$和$x_2$之前的可选字符，还更有希望能匹配成功。

如此看来，我们初始化两个指针i和j，分别指向t和s的开头：
1. 每次贪心地匹配它们指向的字符，如果匹配成功则i和j同时右移，匹配t的下一个位置；
2. 如果匹配失败则j右移，i不变，尝试用s的下一个字符匹配t；
3. 最终如果i移动到t的末尾，则说明t是s的子序列。

第二个问题可以通过便利dictionary中的字符串，并维护当前长度最长且字典序最小的字符串来找到。在实际代码中我们只需要在遍历过程中找到匹配的字符串时和之前的最长的ans比较即可。

## 代码

golang实现：

```go
func findLongestWord(s string, dictionary []string) (ans string) {
	for _, t := range dictionary {
		i := 0
		for j := range s {
			if s[j] == t[i] {
				i++
				if i == len(t) {
					if len(t) > len(ans) || len(t) == len(ans) && t < ans {
						ans = t
					}
					break
				}
			}
		}
	}
	return
}
```

## 时间和空间复杂度

- 时间复杂度：O(d×(m+n))，其中d表示dictionary的长度，m表示s的长度，n表示dictionary中字符串的平均长度。我们需要遍历dictionary中的d个字符串，每个字符串需要O(n+m)的时间复杂度来判断该字符串是否为s的子序列。
- 空间复杂度：O(1)。

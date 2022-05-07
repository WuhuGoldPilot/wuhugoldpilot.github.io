---
layout: article
title: 最小覆盖子串
tags: algorithm double_pointer silding_window
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-07-algorithm-minWindow
comment: true
mermaid: true
chart: true
---

# 题目描述

给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。  
注意：
对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
如果 s 中存在这样的子串，我们保证它是唯一的答案。

示例 1：  
输入：s = "ADOBECODEBANC", t = "ABC"  
输出："BANC"  

示例 2：  
输入：s = "a", t = "a"  
输出："a"  

示例 3:  
输入: s = "a", t = "aa"  
输出: ""  
解释: t 中两个字符 'a' 均应包含在 s 的子串中，因此没有符合条件的子字符串，返回空字符串。  
 

提示：  
$1 <= s.length, t.length <= 105$  
s 和 t 由英文字母组成  
 

**进阶**：你能设计一个在 o(n) 时间内解决此问题的算法吗？

# 解题思路

## 滑动窗口

我们可以使用滑动窗口的思想解决这个问题，在滑动窗口题型中会有两个指针，一个用于延伸现有窗口的r指针，一个用于收缩窗口的l指针。在任意时刻，只有一个指针在运行，另一个保持静止。  
我们在s上滑动窗口，通过移动r指针不断扩张窗口，直至窗口中包含t的所有所需的字符。如果满足要求的窗口能收缩，我们就保持r不动移动l收缩窗口得到最小窗口。

在代码实现中我们需要两个map去记录t的字符（charMap）和滑动窗口出现的字符的个数（countMap），通过check方法去检查当前窗口的countMap是否已包含t所需的所有字符。如果已经包含了t所需的所有字符，我们就保持r指针不动，再缩小l指针看是否能找到更小的窗口，当然在缩小滑动窗口的时候我们要始终满足l<=r的前提条件。

# 代码

golang实现：
```go
import "math"

func minWindow(s string, t string) string {
	charMap, countMap := map[byte]int{}, map[byte]int{}
	tLen := len(t)
	// 记录t的字符
	for i := 0; i < tLen; i++ {
		charMap[t[i]]++
	}
	sLen := len(s)
	// 记录最小结果长度
	len := math.MaxInt32
	ansL, ansR := -1, -1
	// 检查滑动窗口包含t所有字符
	check := func() bool {
		for k, v := range charMap {
			if countMap[k] < v {
				return false
			}
		}
		return true
	}
	// 这里两层循环可以看作r移动时l不移动，l不移动时r不移动，所以实际上还是遍历一遍
	for l, r := 0, 0; r < sLen; r++ {
		if r < sLen && charMap[s[r]] > 0 { // 记录在t中且在s出现的字符
			countMap[s[r]]++
		}
		for check() && l <= r { // 所有t的字符都在滑动窗口内了，缩小len
			if r-l+1 < len { // 小于最小len时更新len
				len = r - l + 1
				ansL, ansR = l, l+len
			}
			if _, ok := charMap[s[l]]; ok { // 逐渐扩大l，观察是否可以缩短len
				countMap[s[l]]--
			}
			l++
		}
	}
	if ansL == -1 {
		return ""
	}
	return s[ansL:ansR]
}
```

# 时间和空间复杂度

- 时间复杂度：最坏情况下左右指针对s的每个元素各遍历一遍，哈希表中对s中的每个元素各插入、删除一次，对t中的元素各插入一次。每次检查是否可行会遍历整个t的哈希表，哈希表的大小与字符集的大小有关，设字符集大小为C，则渐进时间复杂度为O(C⋅∣s∣+∣t∣)。
- 空间复杂度：这里用了两张哈希表作为辅助空间，每张哈希表最多不会存放超过字符集大小的键值对，我们设字符集大小为 C ，则渐进空间复杂度为 O(C)。


---
layout: article
title: 划分字母区间
tags: algorithm greedy 
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-20-algorithm-partitionLabels
comment: true
mermaid: true
chart: true
---

# 题目描述

字符串 S 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。返回一个表示每个字符串片段的长度的列表。  

示例：

输入：S = "ababcbacadefegdehijhklij"

输出：[9,7,8]

解释：
划分结果为 "ababcbaca", "defegde", "hijhklij"。每个字母最多出现在一个片段中。像 "ababcbacadefegde", "hijhklij" 的划分是错误的，因为划分的片段数较少。

提示：
- S的长度在[1, 500]之间。
- S只包含小写字母 'a' 到 'z' 。
 

# 解题思路

## 贪心解法

为了满足贪心策略，我们通常需要一些预处理，其他几遍贪心策略解决的算法的文章出现排序，这属于一种预处理。但是还有很多其他的预处理，例如在处理数组前，统计一遍信息（如频率、个数、第一次出现的位置、最后一次出现的位置等），可以让题目变得简单明了。

这道题我们先遍历一遍字符串得到每个字母的最后一次出现的位置，然后使用贪心策略将字符串划分为尽可能多的片段，具体做法如下：
1. 从左到右遍历字符串，记录当前维护片段的开始下表start和end，初始化时start=end=0；
2. 对于每个出现的字母c，将预处理得到的这个字母的最后出现的下标和当前维护片段end比较，可以知道当前片段的end一定不会小于这个字母的最后出现的下标，所以如果当前出现字母的最后出现的下标大于当前维护片段的end，更新end为end = lastPos[v-'a']；
3. 当访问到end时，当前偏大u那你呢访问结束，片段的长度为$end-start+1$，将该长度添加到结果，更新$start=end+1$继续寻找下一个片段；
4. 重复上述1-3步直至遍历完字符串。

上述做法贪心的思想体现在寻找每个片段可能的最小结束下标，可以保证每个片段的长度一定是满足要求的最短长度，如果取更短长度，就会出现同个字母出现在多个片段中。由于每个片段的结束标志是访问到下表end，因此可以保证每个片段的每个字母一定在当前片段内，不可能出现在其他片段，保证同一个出现在一个片段内的条件成立。

# 代码

golang实现：
```go
func partitionLabels(s string) (ans []int) {
	lastPos := [26]int{}
	for i, v := range s {
		lastPos[v-'a'] = i
	}
	var start, end int
	for i, v := range s {
		if lastPos[v-'a'] > end {
			end = lastPos[v-'a']
		}
		if i == end {
			ans = append(ans, end-start+1)
			start = end + 1
		}
	}
	return
}
```

# 时间和空间复杂度

- 时间复杂度：$O(n)$，其中 $n$ 是字符串的长度。需要遍历字符串两次，第一次遍历时记录每个字母最后一次出现的下标位置，第二次遍历时进行字符串的划分。
- 空间复杂度：$O(∣Σ∣)$，其中 $Σ$ 是字符串中的字符集。这道题中，字符串只包含小写字母，因此$∣Σ∣=26$。

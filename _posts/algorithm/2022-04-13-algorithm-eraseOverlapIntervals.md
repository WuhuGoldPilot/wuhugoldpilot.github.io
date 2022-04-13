---
layout: article
title: 无重叠区间
tags: algorithm greedy interval
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-13-algorithm-eraseOverlapIntervals
comment: true
mermaid: true
chart: true
---

# 题目描述

给定一个区间的集合 intervals ，其中 intervals[i] = [starti, endi] 。返回
需要移除区间的最小数量，使剩余区间互不重叠 。  

示例 1:  

输入: intervals = [ [1,2],[2,3],[3,4],[1,3] ]  
输出: 1  
解释: 移除 [1,3] 后，剩下的区间没有重叠。  

示例 2:  

输入: intervals = [ [1,2], [1,2], [1,2] ]  
输出: 2  
解释: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。  

示例 3:  

输入: intervals = [ [1,2], [2,3] ]  
输出: 0  
解释: 你不需要移除任何区间，因为它们已经是无重叠的了。  

提示:  

$1 <= intervals.length <= 10^5$  
$intervals[i].length == 2$  
$-5 * 10^4 <= starti < endi <= 5 * 10^4$  

# 解题思路

## 贪心解法

求最少的移除区间的个数，等价于尽量保留多的不重叠的区间。要保留多的区间，区间的结尾越小，余留给其他区间的空间就越大。所以我们的贪心策略为：优先保留结尾小的不相交的区间。

具体实现：我们可以将区间按照结尾大小进行从小到大排序，每次选择结尾最小的且和前一个选择的区间不重叠的区间保留。在实际实现中，我们使用一个right变量记录上一次保留的区间的结尾。在遍历排好序的区间时将rigth和当前区间的开头比较：
1. 如果当前区间的开头比上一个保留的区间结尾rigth小，则该区间是要丢弃的；
2. 如果当前区间的开头大于或等于上一个保留的区间结尾right，则该区间保留，且更新rigth为当前区间的结尾。

# 代码

golang实现：
```go
import (
	"sort"
)

func eraseOverlapIntervals(intervals [][]int) int {
	n := len(intervals)
	if n == 0 {
		return 0
	}
	sort.Slice(intervals, func(i, j int) bool {
		return intervals[i][1] < intervals[j][1]
	})
	ans, right := 0, intervals[0][1]
	for i := 1; i < n; i++ {
		if intervals[i][0] < right {
			ans++
		} else {
			right = intervals[i][1]
		}
	}
	return ans
}
```

# 时间和空间复杂度

- 时间复杂度：$O(nlogn)$，其中$n$是区间的数量。我们需要$O(nlogn)$的时间对所有的区间按照右端点进行升序排序，并且需要$O(n)$的时间进行遍历。由于前者在渐进意义下大于后者，因此总时间复杂度为$O(nlogn)$。
- 空间复杂度：$O(logn)$，即为排序需要使用的栈空间。

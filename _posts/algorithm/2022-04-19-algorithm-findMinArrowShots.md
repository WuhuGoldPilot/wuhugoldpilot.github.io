---
layout: article
title: 用最少数量的箭引爆气球
tags: algorithm greedy interval
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-19-algorithm-findMinArrowShots
comment: true
mermaid: true
chart: true
---

# 题目描述

有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组points，其中points[i] = [xstart, xend]表示水平直径在 xstart 和 xend之间的气球。你不知道气球的确切 y 坐标。一支弓箭可以沿着 x 轴从不同点完全垂直地射出。在坐标 x 处射出一支箭，若有一个气球的直径的开始和结束坐标为xstart，xend且满足xstart ≤ x ≤ xend，则该气球会被引爆。可以射出的弓箭的数量没有限制。弓箭一旦被射出之后，可以无限地前进。给你一个数组 points ，返回引爆所有气球所必须射出的最小弓箭数。  

示例 1：  

输入：points = [[10,16],[2,8],[1,6],[7,12]]  

输出：2  

解释：气球可以用2支箭来爆破:  
- 在x = 6处射出箭，击破气球[2,8]和[1,6]。
- 在x = 11处发射箭，击破气球[10,16]和[7,12]。

示例 2：   
输入：points = [[1,2],[3,4],[5,6],[7,8]]  

输出：4  

解释：每个气球需要射出一支箭，总共需要4支箭。  

示例 3：  

输入：points = [[1,2],[2,3],[3,4],[4,5]]  

输出：2  

解释：气球可以用2支箭来爆破:  
- 在x = 2处发射箭，击破气球[1,2]和[2,3]。
- 在x = 4处射出箭，击破气球[3,4]和[4,5]。
 
提示:  
$1 <= points.length <= 10^5$
$points[i].length == 2$
$-2^{31} <= xstart < xend <= 2^{31} - 1$

# 解题思路

## 贪心解法

这一题和我们之前做的无重叠区间非常像，但是又有一点不像，仔细读题目应该可以看出。没错，其实这道题可以理解为找到重叠区间的个数。  
我们需要将数组`按右边界升序排序`，然后使用一个right变量保存前面区间的最大的右边界，遍历过程中判断每个区间的左端点和前面可以用一支箭穿破的最小右边界的大小：
1. 如果当前区间的左端点大于前面的最小右端点，则当前区间和前面的区间不重合，需要加一支箭，且将可以用一支箭穿破的气球的最小右边界right更新为当前区间的右边界；
2. 否则证明前一支箭可以和前面的最小右边界的气球用同一支箭穿破；

这里的贪心策略体现在对于遍历到的每一个区间，我们判断它是否可以被前一支箭穿破。如何判断一支箭可以穿破前面的气球和当前的气球呢？因为我们的数组是按右边界升序排序的，所以遍历开始前我们初始化的这个right是当前所有区间中最小的右区间，对于每一个区间我们比较它的左区间是否大于前面最小的右区间即可判断出是否可以被同一支箭穿破。

# 代码

golang实现：
```go
import (
	"sort"
)

func findMinArrowShots(points [][]int) int {
	n := len(points)
	if n <= 0 {
		return 0
	}
	sort.Slice(points, func(i, j int) bool {
		return points[i][1] < points[j][1]
	})
	ans, right := 1, points[0][1]
	for _, v := range points {
		if v[0] > right {
			ans++
			right = v[1]
		}
	}
	return ans
}
```

# 时间和空间复杂度

- 时间复杂度：$O(nlogn)$，其中$n$是区间的数量。我们需要$O(n)$的时间进行遍历数组，但是因为排序最佳时间复杂度为$O(nlogn)$，所有最终时间复杂度为$O(nlogn)$。
- 空间复杂度：$O(logn)$，即为排序需要使用的栈空间。

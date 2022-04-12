---
layout: article
title: 分发糖果
tags: algorithm greedy allocation
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-12-algorithm-candy
comment: true
mermaid: true
chart: true
---

# 题目描述

n 个孩子站成一排。给你一个整数数组 ratings 表示每个孩子的评分。
你需要按照以下要求，给这些孩子分发糖果：

1. 每个孩子至少分配到 1 个糖果。
2. 相邻两个孩子评分更高的孩子会获得更多的糖果。


请你给每个孩子分发糖果，计算并返回需要准备的 最少糖果数目 。

示例 1：

输入：ratings = [1,0,2]
输出：5

解释：你可以分别给第一个、第二个、第三个孩子分发 2、1、2 颗糖果。

示例 2：
输入：ratings = [1,2,2]
输出：4

解释：你可以分别给第一个、第二个、第三个孩子分发 1、2、1 颗糖果。第三个孩子只得到 1 颗糖果，这满足题面中的两个条件。

提示：  
$n = ratings.length$  
$1 <= n <= 2 * 10^4$  
$0 <= ratings[i] <= 2 * 10^4$    


# 解题思路

这题属于贪心算法的题型，根据上一题的`分发饼干`，你可能会认为这种题型一定是排序？事实上虽然这一题运用了贪心策略，但是我们只需要两次遍历就可以了：
1. 先把所有孩子的糖果初始化为1
2. 从左到右遍历一边，如果右边孩子比左边孩子评分高，且左边孩子不大于右边孩子糖果，则右边孩子的糖果数为左边孩子糖果数$+1$;
3. 从右到左遍历一边，如果左边孩子比右边孩子评分高且糖果数比他少，则左边孩子的糖果数为右边孩子糖果数$+1$；

在这一题的解法中，贪心策略为：每次遍历中只考虑并更新相邻一侧的大小关系。

# 代码

golang实现：
```go
func candy(ratings []int) int {
	n := len(ratings)
	if n <= 0 {
		return 0
	}
	candy := make([]int, len(ratings))
	for i := 0; i < n; i++ {
		candy[i] = 1
	}
	for i := 1; i < n; i++ {
		if ratings[i] > ratings[i-1] {
			candy[i] = candy[i-1] + 1
		}
	}
	for i := n - 2; i >= 0; i-- {
		if ratings[i] > ratings[i+1] && candy[i] <= candy[i+1] {
			candy[i] = candy[i+1] + 1
		}
	}
	var ans int
	for _, v := range candy {
		ans += v
	}
	return ans
}
```

# 时间和空间复杂度

- 时间复杂度：$O(n)$，其中$n$是孩子的数量。我们需要遍历两次数组以分别计算满足左规则或右规则的最少糖果数量。
- 空间复杂度：$O(n)$，其中$n$是孩子的数量。我们需要保存所有的左右规则对应的糖果数量。

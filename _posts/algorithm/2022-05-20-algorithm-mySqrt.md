---
layout: article
title:  x 的平方根
tags: algorithm binary_search 
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-20-algorithm-mySqrt
comment: true
mermaid: true
chart: true
---

## 题目描述

给你一个非负整数x，计算并返回x的算术平方根 。由于返回类型是整数，结果只保留整数部分 ，小数部分将被舍去 。  
注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x ** 0.5 。  

示例 1：  
输入：x = 4  
输出：2  

示例 2：  
输入：x = 8  
输出：2  
解释：8 的算术平方根是 2.82842..., 由于返回类型是整数，小数部分将被舍去。  

提示：  
$0 <= x <= 2^{31} - 1$  

## 解题思路

### 二分搜索

由于x的平方根ans的整数部分满足$k^2<=x$的最大k值，因此我们可以对k进行二分查找，就能得到答案。
二分查找的下界为0，上界可以粗略弟设为x。在二分查找的每一步中，，我们只需要比较中间元素mid的平方与x的大小关系，并通过结果调整上下界的范围。由于我们的运算都是整数运算，所以得到最终答案后，不需要尝试${ans+1}^2$是否大于x了。
代码实现中最后return r是因为当前l已经越过r了，所以r就是最符合答案的整数了，不理解可以使用示例2去演练一遍代码。

## 代码

golang实现：

```go
func mySqrt(x int) int {
	if x == 0 {
		return x
	}
	l, r := 1, x
	for l <= r {
		mid := l + (r-l)/2
		temp := mid * mid
		if temp == x {
			return mid
		} else if temp > x {
			r = mid - 1
		} else {
			l = mid + 1
		}
	}
	return r
}
```

## 时间和空间复杂度

- 时间复杂度：时间复杂度：O(logx)，即为二分查找需要的次数。
- 空间复杂度：O(1)。

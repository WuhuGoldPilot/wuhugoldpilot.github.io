---
layout: article
title: 两数之和 II - 输入有序数组
tags: algorithm double_pointer two_sum
sharing: true
show_author_profile: true
typora-copy-images-to: ..\..\assets\blog_img
typora-root-url: ..\..
key: 2022-04-26-algorithm-twoSum-ii
comment: true
mermaid: true
chart: true
---

# 题目描述

给你一个下标从1开始的整数数组numbers，该数组已按非递减顺序排列，请你从数组中找出满足相加之和等于目标数target的两个数。如果设这两个数分别是numbers[index1]和numbers[index2]，则$1 <= index1 < index2 <=numbers.length$。以长度为2的整数数组[index1, index2]的形式返回这两个整数的下标index1和index2。  
你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。 你所设计的解决方案必须只使用常量级的额外空间。
 
示例 1：  

输入：numbers = [2,7,11,15], target = 9  
输出：[1,2]  
解释：2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。  
 
示例 2：  

输入：numbers = [2,3,4], target = 6  
输出：[1,3]  
解释：2 与 4 之和等于目标数 6 。因此 index1 = 1, index2 = 3 。返回 [1, 3] 。  
 
示例 3：  
输入：numbers = [-1,0], target = -1  
输出：[1,2]  
解释：-1 与 0 之和等于目标数 -1 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。  

提示：  

$2 <= numbers.length <= 3*10^4$  
$-1000 <= numbers[i] <= 1000$  
numbers 按 非递减顺序 排列  
$-1000 <= target <= 1000$
仅存在一个有效答案  

# 解题思路

## 双指针

首先我们需要明确题目中给出的条件；
1. 数组是排序的；
2. 不可以使用重复的相同元素；
3. 只能使用常数空间。

因为只能使用常数空间，而且数组又是排序的，这个题目又是检索的题目，我们很容易就想到双指针这种解法。  
具体做法为：
1. 首先初始化左右两个指针分别为数组开头和结尾；
2. 将它们的和和target比较：
   1. 如果等于target直接返回结果；
   2. 如果大于target，证明right指针偏右，需要左移右指针缩小两者之和，左指针保持不变；
   3. 如果小于target，证明left指针偏左，需要右移左指针扩大两者之和，右指针保持不变。

在实际代码中，当左右指针指向的数字之和不等于target时，我们在移动左右指针的时候理应是当前移动的指针移到不等于需要移动的指针指向的数字的位置。意思就是移动left的时候我们要移动到大于numbers[left]的位置才进行下一轮比较；同理移动right的时候也一样，我们需要移动到小于numbers[right]的位置，当然移动的时候也需要同时满足$left<right$这个前提。

# 代码

golang实现：
```go
func twoSum(numbers []int, target int) (ans []int) {
	left, right := 0, len(numbers)-1
	for left < right {
		sum := numbers[left] + numbers[right]
		if sum == target {
			ans = append(ans, left+1, right+1)
			return
		} else if sum > target {
			for right--; right > left && numbers[right] == numbers[right+1]; right-- { // right左移到小于当前比较的numbers[right]的位置且不与left相交
			}
		} else {
			for left++; left < right && numbers[left] == numbers[left-1]; left++ { // left右移到大于当前比较的numbers[left]的位置且不与right相交
			}
		}
	}
	return
}
```

# 时间和空间复杂度

- 时间复杂度：$O(n)$，其中n是数组的长度，两个指针移动的总次数最多为n次。
- 空间复杂度：$O(1)$。

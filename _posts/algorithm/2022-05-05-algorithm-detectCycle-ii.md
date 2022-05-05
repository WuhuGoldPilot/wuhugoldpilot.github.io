---
layout: article
title: 环形链表 II
tags: algorithm double_pointer
sharing: true
show_author_profile: true
typora-copy-images-to: ../../assets/blog_img
typora-root-url: ../..
key: 2022-05-05-algorithm-detectCycle-ii
comment: true
mermaid: true
chart: true
---

# 题目描述

给定一个链表的头节点head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。
如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。
不允许修改链表。

示例 1：  
![detectCycle_1](/assets/blog_img/detectCycle_1.png)  
输入：head = [3,2,0,-4], pos = 1  
输出：返回索引为 1 的链表节点  
解释：链表中有一个环，其尾部连接到第二个节点。  


示例 2：  
![detectCycle_2](/assets/blog_img/detectCycle_2.png)  
输入：head = [1,2], pos = 0  
输出：返回索引为 0 的链表节点  
解释：链表中有一个环，其尾部连接到第一个节点。  


示例 3：  
![detectCycle_3](/assets/blog_img/detectCycle_3.png)  
输入：head = [1], pos = -1  
输出：返回 null  
解释：链表中没有环。  
 

提示：  
链表中节点的数目范围在范围 [0, 104] 内  
-105 <= Node.val <= 105  
pos 的值为 -1 或者链表中的一个有效索引  

**进阶**：你是否可以使用 O(1) 空间解决此题？

# 解题思路

## 双指针

我们使用两个指针fast和slow，它们起始都位于链表的头部。然后，slow每次向后移动一个位置，fast每次向后移动两个位置。如果链表中有环，那么fast指针会与slow指针在环中相遇。  
如下图所示：
![detectCycle](/assets/blog_img/detectCycle.png)  
设链表中环外部分长度为a，slow指针进入环后，走了b的距离和fast相遇。此时，fast已经走完了环的n圈，因此fast走过的总距离为：$a+n(b+c)+b=a+(n+1)b+nc$。
> 上面推导有一个隐含假设条件藏在“slow指针进入环后，走了b的距离和fast相遇”，也就是说slow一定会在走第一圈中和fast相遇。为什么slow肯定会和fast在slow走的第一圈相遇呢？
> 1. fast先进入环，当slow到达环的入口时，fast此时在环中的某个位置（有可能也在环的入口），设此时快指针和慢指针的距离为x；
> 2. 若fast此时也在环的入口，则x=0，这种情况肯定在第一圈内相遇，下面我们讨论一般情况；
> 3. 设环的周长为n，那么可以把当前情况看成快指针追赶满指针，需要追赶的长度为n-x；
> 4. fast的速度是slow的两倍，那么追赶n-x的距离需要n-x次移动fast就能追上slow；
> 5. 在n-x次移动内，slow走了n-x步，因为x>=0，则慢指针走的路程少于等于n，即走不完一圈就和fast相遇。

根据题意，任意时刻，fast走过的距离都是slow的两倍，因此：
$$a+(n+1)b+nc=2(a+b) => a=c+(n-1)(b+c)$$
有了$a=c+(n-1)(b+c)$的等量关系，我们可以发现，从相遇点到入环点到距离c加上n-1圈的环长，恰好等于从链表头部到入环点到距离。因此，当fast和slow相遇时，我们新创建一个指针p指向链表头部，slow和p同时往前走，当它们相遇时就是环的入口。

# 代码

golang实现：
```go
type ListNode struct {
	Val  int
	Next *ListNode
}

func detectCycle(head *ListNode) *ListNode {
	fast, slow := head, head
	for fast != nil {
		slow = slow.Next
		if fast.Next == nil {
			return nil
		}
		fast = fast.Next.Next
		if fast == slow {
			p := head
			for p != slow {
				p = p.Next
				slow = slow.Next
			}
			return p
		}
	}
	return nil
}
```

# 时间和空间复杂度

- 时间复杂度：O(N)，其中N为链表中节点的数目。在最初判断快慢指针是否相遇时，slow指针走过的距离不会超过链表的总长度；随后寻找入环点时，走过的距离也不会超过链表的总长度。因此，总的执行时间为 $O(N)+O(N)=O(N)$。
- 空间复杂度：O(1)。我们只使用了slow, fast, p三个指针。


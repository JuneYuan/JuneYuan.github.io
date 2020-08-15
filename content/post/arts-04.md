---
title: "ARTS (0810-0816)"
date: 2020-08-15T18:52:31+08:00
draft: false
---

# Algorithm

## 链表排序

复杂度 O(n log n). [Leetcode](https://leetcode.com/problems/sort-list/)

### 题解1 - 归并排序(链表长度求中间节点)

可考虑快排/ 堆排/ 归并排序。这里用归并排序。

实现过程：按长度等分链表，断开为左右两截，对其分别排序，最后合并。

细节：

+ 由于用到长度，所以预先求出长度，传参给辅助函数。
+ 链表分为两段，指针必须断开。
+ 处理好递归退出条件。
+ merge 过程处理好指针。

```
// Top-down merge sort. Recursive implementation.
func sortList(head *ListNode) *ListNode {
    len := length(head)
    if len <= 1 { return head }

    return mergeSort(head, len)
}

func length(head *ListNode) int {
	var len int
	for p := head; p != nil; p = p.Next {
		len++
	}
	return len
}

func mergeSort(head *ListNode, len int) *ListNode {
	fmt.Printf("mergeSort(%v, %v)\n", head, len)
	if len <= 1 { return head }

	var (
		lenL = (len + 1)/2
		lenR = len - lenL
		headL = head
		headR *ListNode
		dummy = &ListNode{Next: head}
		p = dummy
	)
	for i := 0; i < lenL; i++ {
		p = p.Next
	}
	headR = p.Next
	p.Next = nil // break the link between left half and right half
	fmt.Printf("headL=%v headR=%v\n", headL, headR)

	headL = mergeSort(headL, lenL)
	headR = mergeSort(headR, lenR)

	return merge2(headL, headR)
}

func merge2(headL, headR *ListNode) *ListNode {
	var dummy = &ListNode{}
	var pL, pR, p = headL, headR, dummy

	for ; pL != nil && pR != nil; {
		if pL != nil && pR != nil {
			if pL.Val < pR.Val {
				p.Next = pL
				pL = pL.Next
			} else {
				p.Next = pR
				pR = pR.Next
			}
			p = p.Next
		}
	}

	if pL != nil {
		p.Next = pL
	} else {
		p.Next = pR
	}

	return dummy.Next
}
```

### 题解2 - 归并排序(快慢指针求中间节点)

等分链表也可以不求长度，改为快慢指针来找中间节点。

细节：快慢指针的初始化，可以都用 dummy 节点，也可以 slowPtr=head, fastPtr=head.Next. 纸笔举例分析可知两种方式等效。

```
func sortList_fastSlowPtr(head *ListNode) *ListNode {
	fmt.Printf("sortList_fastSlowPtr(%v)\n", head)
	if head == nil || head.Next == nil { return head }

	var (
		headL = head
		headR *ListNode
		dummy = &ListNode{Next: head}
		fastPtr, slowPtr = dummy, dummy
	)

	for fastPtr != nil && fastPtr.Next != nil {
		fastPtr = fastPtr.Next.Next
		slowPtr = slowPtr.Next
	}
	headR = slowPtr.Next
	slowPtr.Next = nil
	fmt.Printf("headL=%v headR=%v\n", headL, headR)

	headL = sortList_fastSlowPtr(headL)
	headR = sortList_fastSlowPtr(headR)

	return merge2(headL, headR)
}
```

### 题解3 - 归并排序(自底向上)

初始时把链表视为一个个孤立的节点，每个节点都是长度为1的链表。每一轮归并相邻两子链表，经过 log(n) 轮后得到最终的有序链表。

以 `-1->5->3->4->0` 为例，每轮归并后的结果依次为：

```
subList=[-1 5 3 4 0]
subList=[-1->5 3->4 0]
subList=[-1->3->4->5 0]
subList=[-1->0->3->4->5]
```

细节：注意适时断开 link.

```
func sortList_bottomUp(head *ListNode) *ListNode {
	if head == nil || head.Next == nil { return head }

	// put all list nodes in a slice
	var subList []*ListNode
	for p := head; p != nil; {
		x := p
		p = p.Next
		x.Next = nil
		subList = append(subList, x)
	}

	// merge every 2 elements in subList
	for ; len(subList) > 1; {
		fmt.Printf("subList=%v\n", subList)
		var subList1 []*ListNode
		for i := 0; i < len(subList); i+=2 {
			if i + 1 >= len(subList) {
				subList = append(subList, nil) // ensure subList has even number of elements
			}
			subList1 = append(subList1, merge2(subList[i], subList[i+1]))
		}
		subList = subList1
		subList1 = subList1[:0]
	}
	fmt.Printf("subList=%v\n", subList)

	return subList[0]
}
```



---
title: 单链表求倒数第k个节点
tags:
  - 数据结构
  - 算法
  - 线性表
abbrlink: bff76920
date: 2024-04-16 23:51:31
---


# 题目

带头节点的链表, 给出了头指针`list`, 设计一个高效的算法, 找到倒数第`k`个节点.

如果找到返回`true`, 并打印倒数第`k`个节点的数据, 否则返回`false`.

# 分析

使用双指针一前一后, 当作游标同时偏移. 前一个指针刚好领先后一个指针`k`个位置. 当前一个指针偏移到链表末尾时, 后一个指针刚好指向倒数第`k`个节点.


# 实现

```c
ElemType Sreach_K(LinkList list, ElemType k)
{
	int count = 0;
	LNode *p = list->next;
	LNode *q = list->next;
	while (p != NULL)
	{
		if (count < k)
			count++;
		else
			q = q->next;
		p = p->next;
	}
	if (count < k)
		return 0;
	printf("%d", q->data);
	return 1;
}
```








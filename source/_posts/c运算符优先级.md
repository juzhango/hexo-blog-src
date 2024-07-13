---
title: c运算符优先级——持续更新
abbrlink: 64afdbc1
date: 2024-07-13 17:23:59
tags: c语言
hidden: true
---







<table>
	<!-- 1 -->
	<tr>
		<td>优先级</td>
		<td>运算符</td>
		<td>名称和含义</td>
		<td>结合方向</td>
		<td>说明</td>
	</tr>
	<tr>
		<td rowspan=5 colspan=1>1</td>
		<td rowspan=1 colspan=1>[]</td>
		<td rowspan=1 colspan=1>数组下标</td>
		<td rowspan=5 colspan=1>左到右</td>
		<td rowspan=5 colspan=1>--</td>
	</tr>
	<tr>
	<tr>
		<td rowspan=1 colspan=1>()</td>
		<td rowspan=1 colspan=1>圆括号</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>.</td>
		<td rowspan=1 colspan=1>结构体成员选择</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>-></td>
		<td rowspan=1 colspan=1>结构体成员选择</td>
	</tr>
	<!-- 2 -->
	<tr>
		<td rowspan=9 colspan=1>2</td>
		<td rowspan=1 colspan=1>-</td>
		<td rowspan=1 colspan=1>负号</td>
		<td rowspan=9 colspan=1>右到左</td>
		<td rowspan=7 colspan=1>单目运算符</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>~</td>
		<td rowspan=1 colspan=1>按位取反</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>++</td>
		<td rowspan=1 colspan=1>自增</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>--</td>
		<td rowspan=1 colspan=1>自减</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>*</td>
		<td rowspan=1 colspan=1>指针解引用</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>&</td>
		<td rowspan=1 colspan=1>取地址</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>!</td>
		<td rowspan=1 colspan=1>逻辑非</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>(tpye)</td>
		<td rowspan=1 colspan=1>强制类型转换</td>
		<td rowspan=2 colspan=1>--</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>sizeof</td>
		<td rowspan=1 colspan=1>长度运算符</td>
	</tr>
	<!-- 3 -->
	<tr>
		<td rowspan=3 colspan=1>3</td>
		<td rowspan=1 colspan=1>/</td>
		<td rowspan=1 colspan=1>除</td>
		<td rowspan=3 colspan=1>左到右</td>
		<td rowspan=3 colspan=1>双目运算符</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>*</td>
		<td rowspan=1 colspan=1>除</td>
	</tr>
	<tr>
		<td rowspan=1 colspan=1>%</td>
		<td rowspan=1 colspan=1>取模</td>
	</tr>
</table></table>

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240713/20240713200001.png)

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240713/20240713200035.png)


![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240713/20240713200048.png)


# 例子

## *p++

`*`和`++` 属于同一优先级运算符, 结合方向是从右至左, 所以这个表达式为: 将指针`p`加1后, 再进行指针解引用.






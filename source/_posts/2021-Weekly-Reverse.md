---
title: 2021 Weekly Reverse
date: 2021-02-01 21:04:56
excerpt: 每周的逆向练习与 Writeup
tags: 
- Reverse
- BUUCTF
---

## Week 5, 01/31 - 02/06, BUUCTF

## reverse1

用 ExeinfoPE 看一下, 是一个 x64 程序.

![image-20210201212113576](2021-Weekly-Reverse/image-20210201212113576.png)

IDA, 入口是`sub_140012170`, 一路跟踪下去到`sub_140012190`,  ~~然后分析不能~~

从 Strings window 看一下, 发现一条明显的提示语 `wrong flag`, 看一下, 是`sub_1400118C0`引用了它, 跳过去康康.

![image-20210201212247636](2021-Weekly-Reverse/image-20210201212247636.png)

第18行的 `for`循环中, 把 `Str2[]`中的`o`全部替换为了`0`, 然后和输入的`Str1`进行比较, 判断是否正确.

`Str2[]`的内容是`{hello_world}`, 替换后输入程序检查, 确定正确.

`flag{hell0_w0rld}`




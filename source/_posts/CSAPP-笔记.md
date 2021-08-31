---
title: CSAPP 第三章 笔记
date: 2021-08-31 21:41:22
excerpt: CSAPP 第三章的笔记, 包括初级汇编
tags:
- 读书笔记
- CSAPP
---

> CSAPP 采用的是 x64 ATT 汇编, 然而平时用的 IDA 什么的都是 MASM 汇编... 有点头大
> 话说回来, 都会肯定不是坏事... 大概

# 编译 汇编与反汇编

## 编译

`gcc -Og -o [output file] [source files]`

包括四个步骤: 预处理, 编译, 汇编, 链接

- **预处理**
  - 把 `#` 开头的预处理指令展开, 比如头文件展开, 宏展开
  - 处理完了也是 `*.c` 或者 `*.cpp`
  - `gcc -E [source files] -o [output file]`
- **编译**
  - 把预处理完的源代码转化成汇编代码
  - 对于 `gcc`, 默认是 ATT 格式, 用 `-masm=intel` 指定采用 Intel 格式
  - `gcc -S <source>.c -o <output>.s`
- **汇编**
  - 把汇编代码转为二进制的目标文件
  - `gcc -c <source>.c -o <output>.o`
- **链接**
  - 将该目标文件与其他目标文件、库文件、启动文件等链接起来生成可执行文件
  - `gcc <object>.o -o <executable>`

## 反汇编

用 `objdump`

- `-d <file(s)>`: 将代码段反汇编；
- `-S <file(s)>`: 将代码段反汇编的同时，将反汇编代码与源代码交替显示，编译时需要使用-g- 参数，即需要调试信息；
- `-C <file(s)>`: 将C++符号名逆向解析
- `-l <file(s)>`: 反汇编代码中插入文件名和行号
- `-j section <file(s)>`: 仅反汇编指定的section

# 汇编语言 - 初步

## 数据格式

> 为什么一个字是两字节?
>  
> 实际上, 一个字并不一定是两个字节 - 比如 ARM 的 NEON 指令集下, 一个字就是 32 位 (四字节). 但是在 x86/x64 环境下, 一个字规定为两字节, 从 8086 开始就是这样了
> 另外, 一个字节也不一定是 8 位...
>
>Reference: [StackOverflow](https://stackoverflow.com/questions/28066462/how-many-bits-is-a-word)

| 名称 | 长度 (字节) | ATT格式后缀 | MASM中的类型 |
| ---- | ----------- |----------- | ------------ |
| 字节 | 1           | b           | BYTE         |
| 字  | 2           | w           | WORD         |
| 双字 | 4           | l           | DWORD        |
| 四字 | 8           | q           | QWORD        |

## 关于 LEA 指令

`lea` 指令名为 "加载有效地址" (load effective address), 实际上也可以进行简单的四则运算 (利用那些繁杂但必要的寻址方式).

相对于使用多条`add` `sub`指令, 简单的四则运算用 `lea` 显然更便捷 - 这个时候和地址计算就没任何关系了

> 这点坑了我半天

## 条件判断

### CMP 指令

和 `SUB` 进行的运算一样, 但不改变操作数寄存器的值

### TEST 指令

和 `AND` 进行的运算一样, 但不改变操作数寄存器的值

所以 `test %rax, %rax` 这类两个操作数相同的指令可以用来**判断其值的正负**

## switch 的实现

在分支较多且数值较为接近的时候**可能**会采用跳转表

# 过程控制

//TODO

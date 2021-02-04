---
title: BUUCTF Weekly Reverse
date: 2021-02-01 21:04:56
excerpt: BUUCTF上的逆向练习与 Writeup
tags: 
- Reverse
- BUUCTF
---

## Week 5, 01/31 - 02/06, BUUCTF

### reverse1

用 ExeinfoPE 看一下, 是一个 x64 程序.

![image-20210201212113576](2021-Weekly-Reverse/image-20210201212113576.png)

IDA, 入口是`sub_140012170`, 一路跟踪下去到`sub_140012190`,  ~~然后分析不能~~

从 Strings window 看一下, 发现一条明显的提示语 `wrong flag`, 看一下, 是`sub_1400118C0`引用了它, 跳过去康康.

![image-20210201212247636](2021-Weekly-Reverse/image-20210201212247636.png)

第18行的 `for`循环中, 把 `Str2[]`中的`o`全部替换为了`0`, 然后和输入的`Str1`进行比较, 判断是否正确.

`Str2[]`的内容是`{hello_world}`, 替换后输入程序检查, 确定正确.

`flag{hell0_w0rld}`

### reverse3

IDA, 找到 `main`之后 F5, 大概修改一些名称之后得到如下.

```c
int __cdecl main_0(int argc, const char **argv, const char **envp)
{
  size_t StrLen; // eax
  const char *v4; // eax
  size_t destLen2; // eax
  char v7; // [esp+0h] [ebp-188h]
  char v8; // [esp+0h] [ebp-188h]
  signed int j; // [esp+DCh] [ebp-ACh]
  int i; // [esp+E8h] [ebp-A0h]
  signed int destLen; // [esp+E8h] [ebp-A0h]
  char Destination[108]; // [esp+F4h] [ebp-94h] BYREF
  char Str[28]; // [esp+160h] [ebp-28h] BYREF
  char v14[8]; // [esp+17Ch] [ebp-Ch] BYREF

  for ( i = 0; i < 100; ++i )
  {
    if ( (unsigned int)i >= 0x64 )
      j____report_rangecheckfailure();
    Destination[i] = 0;
  }
  alert("please enter the flag:", v7);
  input("%20s", (char)Str);
  StrLen = j_strlen(Str);
  v4 = (const char *)sub_4110BE(Str, StrLen, v14);
  strncpy(Destination, v4, 0x28u);
  destLen = j_strlen(Destination);
  for ( j = 0; j < destLen; ++j )
    Destination[j] += j;
  destLen2 = j_strlen(Destination);
  if ( !strncmp(Destination, aE3nifih9bCNDh, destLen2) )
    alert("rigth flag!\n", v8);
  else
    alert("wrong flag!\n", v8);
  return 0;
}
```

在得到输入之后, `sub_4110BE`对输入的 `Str`进行了一些处理, 同时 27 行的 `for` 循环也对 `Destination[]` 进行了移位, 之后两者比较判断是否正确.

观察 `sub_4110BE`, 发现其中调用了一个 base64 的编码表, 结合行为确定是对输入进行 base64 编码.

把 `Destination[]`的内容进行反移位, 得到`e2lfbDB2ZV95b3V9`, base64 解码后得到 `{i_l0ve_you}`

`flag{i_l0ve_you}`

~~另外题中的"right"打错了...~~






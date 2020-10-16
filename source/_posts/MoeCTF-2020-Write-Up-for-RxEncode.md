---
title: MoeCTF 2020 Write Up for RxEncode
date: 2020-10-16 19:32:50
tags: 
- MoeCTF
- Reverse
---

### RxEncode 题解

这道题当时没有做出来 ~~, 然后就变成了面试作业~~

使用IDA反汇编, 观察main函数, 发现程序将输入的内容作为RxEncode函数的参数, 再将返回值s1同s2进行比较确定flag正确与否.

双击RxEncode并反汇编, 得伪代码如下.

``` c
void *__fastcall RxEncode(const char *a1, int a2)
{
  int v2; // eax
  int v3; // edx
  void *result; // rax
  void *Dst; // [rsp+28h] [rbp-58h]
  signed int i; // [rsp+30h] [rbp-50h]
  int v7; // [rsp+34h] [rbp-4Ch]
  int v8; // [rsp+34h] [rbp-4Ch]
  signed int v9; // [rsp+38h] [rbp-48h]
  int v10; // [rsp+3Ch] [rbp-44h]
  _BYTE *v11; // [rsp+40h] [rbp-40h]
  signed int v12; // [rsp+48h] [rbp-38h]
  int v13; // [rsp+4Ch] [rbp-34h]
  const char *v14; // [rsp+70h] [rbp-10h]
  int v15; // [rsp+78h] [rbp-8h]

  v14 = a1;
  v15 = a2;
  v2 = a2;
  v3 = a2 + 3;
  if ( v2 < 0 )
    v2 = v3;
  v13 = 3 * (v2 >> 2);
  v12 = 0;
  v10 = 0;
  if ( a1[v15 - 1] == 61 )
    v12 = 1;
  if ( a1[v15 - 2] == 61 )
    ++v12;
  if ( a1[v15 - 3] == 61 )
    ++v12;
  if ( v12 == 1 )
  {
    v13 += 4;
  }
  else if ( v12 > 1 )
  {
    if ( v12 == 2 )
    {
      v13 += 3;
    }
    else if ( v12 == 3 )
    {
      v13 += 2;
    }
  }
  else if ( !v12 )
  {
    v13 += 4;
  }
  Dst = malloc(v13);
  if ( Dst )
  {
    memset(Dst, 0, v13);
    v11 = Dst;
    while ( v15 - v12 > v10 )
    {
      v9 = 0;
      v7 = 0;
      while ( v9 <= 3 && v15 - v12 > v10 )
      {
        v7 = (v7 << 6) | (char)find_pos(v14[v10]);
        ++v9;
        ++v10;
      }
      v8 = v7 << 6 * (4 - v9);
      for ( i = 0; i <= 2 && i != v9; ++i )
        *v11++ = v8 >> 8 * (2 - i);
    }
    *v11 = 0;
    result = Dst;
  }
  else
  {
    printf("No enough memory.\n");
    result = 0i64;
  }
  return result;
}
```

注意到 '=' 的ASCII码是61, 结合面试时的提示**"将flag进行**(类似)**base64解码"**, 仔细分析RxEncode的逻辑后, 得出这个函数的作用就是将内容进行类似base64的解码.

由此我们得出, s2保存的应当就是解码后的flag内容, 将变量的值转为16进制后结果如下.

``` c
   char s2[8]; // [rsp+10h] [rbp-60h]
  __int64 v15; // [rsp+18h] [rbp-58h]
  __int64 v16; // [rsp+20h] [rbp-50h]

 /*....................................
   其他内容
   .....................................*/

  *(_QWORD *)s2 = 0x4AD158FEB59C879ALL;
  v15 = 0xCBEBFDFA6CED0BFELL;
  v16 = 0x7A47A38E43A334E8LL;
```

结合s2 v15 v16在main的栈帧中相邻存放这一点, 我们推测s2实际的长度应当不止为8, 而是24. 

事实上, 我认为v15 v16的__int64类型也是IDA的推测(它并不确定真实类型是什么) - 因为在栈帧中将s2的类型由默认的char []改为char [24]后, 三个变量变成了这样.

``` c
  *(_QWORD *)s2 = 0x4AD158FEB59C879ALL;
  *(_QWORD *)v15 = 0xCBEBFDFA6CED0BFELL;
  *(_QWORD *)v16 = 0x7A47A38E43A334E8LL;
```

考虑到x86的小端序, 实际的字节顺序应当是 ```9A 87 9C B5 FE 58```这样. 获得解码的flag后, 尝试进行base64编码. 

考虑到flag中含有标准编码表不包含的```{ }```, 因此我们可以认为这个表不同于标准的编码表.

观察发现, RxEncode中调用了一个名为find_pos的函数, 它的作用应当是在编码表中查找字符的位置, 进入该函数, 得到编码表```ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz01234{}789+/=```.

对先前得到的字节按照编码表进行编码, 可以得到flag```moectf{Y0Ur+C+1s+v3ry+g0o0OOo0d}```.

~~No my C is very poooooooor /(ㄒoㄒ)/~~~~
---
title: MoeCTF 2020 Write Up
date: 2020-10-16 17:28:59
tags: 
 - MoeCTF
 - Reverse
 - Misc
 - Algorithm
 - Classical Crypto
 - Android
---

# MoeCTF 2020 Write Up

***By Horizon Chaser, aka. Horizon***

第一次参加CTF, 现学现用, 会做的题实在有限😂, 先把自己会的或者有思路的部分题写下来, 以供参考

~~其他不会的题就等各位巨佬的Write Up辽~~

~~龟速~~更新记录

- 10.14 Misc + Reverse.Rxencode
- 10.16 Reverse
- 10.18 Algorithm + Classical Crypto

## Misc

Misc, 全程Miscellaneous, 本意是"杂项". 在CTF中大概是指多个领域 ~~脑洞~~ 的混合. 因此做起来还是很有意思的.

### Welcome

Misc入门题, 附件profession.jpg. 打开, 发现是专业团队, 仔细一看右下角有黑白色块, 觉得可能是在jpg文件末端添加了内容.

使用binwalk分析, 未发现隐写的文件, 有点疑惑.

使用16进制编辑器打开, 发现flag就在末尾😂

![welcome](MoeCTF%202020%20WriteUp/image-20201012183142831.png)

### hey fxck you!

附件good_morning_my_neighbors.png, 表达了诚挚的问候(大雾)

除了最下方被裁剪了, 图片没有发现什么问题, 使用binwalk分析, 发现末尾有一个zip文件, 解压得fk u.txt, 内容如下

> ++++++++[>>++>++++>++++++>++++++++>++++++++++>++++++++++++>++++++++++++++>++++++++++++++++>++++++++++++++++++>++++++++++++++++++++>++++++++++++++++++++++>++++++++++++++++++++++++>++++++++++++++++++++++++++>++++++++++++++++++++++++++++>++++++++++++++++++++++++++++++<<<<<<<<<<<<<<<<-]>>>>>>>>---.++.<+++++.--.>+++++.<+++.>>-----.--.<<-.>-.<<<<<+.>>>>>>.<<.>.<<<<<.>>>>+.+++++.------------.<+++++.>.<<<++.<.>>>>>>++++.



搜索, 得知这是BrainFuck语言, 解密得到flag ```moectf{yes!yes!fk_U_2!}```

~~这是对此前问候的友好回应(确信)~~





### base64？¿

题面是一个以等号结尾的字符串  ```0H9MJjCNPiMgJHMQJNtfyEJgIjtS1Ig=```, 结合名称, 确定是base64编码的文本.

按照标准的字符表无法解密, 查看hint, 得到字符表为```vwxrstuopq34567ABCDEFGHIJyz012PQRSTKLMNOZabcdUVWXYefghijklmn89+/```, 按照这个字符表解码, 得到flag```moectf{itai_base64_qaq}```



### A3FXCK

~~所以a3又干啥了这是~~

题面是一个jpg, binwalk分析, 得到隐藏的A3FXCK.txt, 打开, 发现内容可被拆分为两类: ```luoqXan```与```arttnbaX```, X为1~6的正整数.

结合首行的```123456[]()+!```, 推测1~6的值分别代表```[]()+!```, 替换后得到一坨奇怪的东西, 尝试按照JavaScript运行后得到flag ```moectf{J5Fxck_1s_1nt3res7in9!}```

新知识: [JSFuck](https://github.com/aemkei/jsfuck)是将(小段的)JS代码加密为仅包含```[]()+!```的文本, 但是会造成严重的体积膨胀. 所以替换后得到的那一坨解密之后也只是一行```alert('moectf{J5Fxck_1s_1nt3res7in9!}'```

~~由此得知, JS确实是最强大的编程语言~~



## Reverse

### Welcome To Re!

签到题, 也是第一道我做出来的逆向的题😂

下载附件, 打开, 按照入门指南和Hint提示, 使用IDA64分析SignIn.exe, 在左侧定位main函数, 双击跳转到对应位置, 按下F5反汇编, 得到伪代码如下

``` c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char Str1; // [rsp+20h] [rbp-60h]
  char Str2[8]; // [rsp+50h] [rbp-30h]

  _main();
  strcpy(Str2, "moectf{W3lc0me-T0_th3-W0rld_Of_R3v3rsE!}");
  puts("Welcome to MoeCTF! --by Reverier\nPlease Input your flag and I will check it:");
  scanf("%41s", &Str1);
  if ( !strcmp(&Str1, Str2) )
    puts("Congratulations!");
  else
    puts("Ruaaaaaaaaaaaaa~~~Wrong!");
  getchar();
  getchar();
  return 0;
}
```

在main函数中, 首先将flag内容复制到str2中, 然后输出提示信息并将输入保存到Str1中. 之后通过strcmp比较输入的Str2与保存的Str1的值是否一致. 由此我们得出, Str2的值就是flag```moectf{W3lc0me-T0_th3-W0rld_Of_R3v3rsE!}```. ~~虽然说是显然的, 但是该分析还是要分析的~~

总之, Welcome to the World of Reverse 🍻!



### RxEncode

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

```C
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

``` C
  *(_QWORD *)s2 = 0x4AD158FEB59C879ALL;
  *(_QWORD *)v15 = 0xCBEBFDFA6CED0BFELL;
  *(_QWORD *)v16 = 0x7A47A38E43A334E8LL;
```

考虑到x86的小端序, 实际的字节顺序应当是 ```9A 87 9C B5 FE 58```这样. 获得解码的flag后, 尝试进行base64编码. 

考虑到flag中含有标准编码表不包含的```{ }```, 因此我们可以认为这个表不同于标准的编码表.

观察发现, RxEncode中调用了一个名为find_pos的函数, 它的作用应当是在编码表中查找字符的位置, 进入该函数, 得到编码表```ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz01234{}789+/=```.

对先前得到的字节按照编码表进行编码, 可以得到flag```moectf{Y0Ur+C+1s+v3ry+g0o0OOo0d}```.

~~No my C is very poooooooor /(ㄒoㄒ)/~~~~

#### 题外话

在分析时我不小心改错了s2的类型, 却又改不回去, 而IDA本身又不支持撤销…
当时我通过菜单栏的New Instance新打开了一个IDA实例重新分析, 得到的main函数中解码后的flag的相关内容变成了这样, 栈帧也发生了相应的改变…

``` c
  Str2 = -98;
  v11 = -101;
  v12 = -100;
  v13 = -75;
  v14 = -2;
  v15 = 112;
  v16 = -45;
  v17 = 15;
  v18 = -78;
  v19 = -47;
  v20 = 79;
  v21 = -100;
  v22 = 2;
  v23 = 127;
  v24 = -85;
  v25 = -34;
  v26 = 89;
  v27 = 101;
  v28 = 99;
  v29 = -25;
  v30 = 64;
  v31 = -99;
  v32 = -51;
  v33 = -6;
  v34 = 4;
  v35 = 0;
  v36 = 0;
  v37 = 0;
  v38 = 0;
  v39 = 0;
  v40 = 0;
  v41 = 0;
  v42 = 0;
```

不过除此之外整体的结构并未发生过多改变, 其余内容一样.



### Simple Re

下载解压, 拖入IDA中反汇编, 得到伪代码. 观察发现, main中会将输入的字符串作为参数调用enc()函数.    

双击跳转进该函数, 反汇编后得到伪代码如下. 

``` c
int __fastcall enc(char *a1)
{
  int result; // eax
    
  /*
  ............
  循环用变量定义
  ............
  */

  for ( i = 0; i <= 30; ++i )
    out[i] = a1[i] ^ 0x17;
  for ( j = 0; j <= 30; ++j )
    out[j] ^= 0x39u;
  for ( k = 0; k <= 30; ++k )
    out[k] ^= 0x4Bu;
  for ( l = 0; l <= 30; ++l )
    out[l] ^= 0x4Au;
  for ( m = 0; m <= 30; ++m )
    out[m] ^= 0x49u;
  for ( n = 0; n <= 30; ++n )
    out[n] ^= 0x26u;
  for ( ii = 0; ii <= 30; ++ii )
    out[ii] ^= 0x15u;
  for ( jj = 0; jj <= 30; ++jj )
    out[jj] ^= 0x61u;
  for ( kk = 0; kk <= 30; ++kk )
    out[kk] ^= 0x56u;
  for ( ll = 0; ll <= 30; ++ll )
    out[ll] ^= 0x1Bu;
  for ( mm = 0; mm <= 30; ++mm )
    out[mm] ^= 0x21u;
  for ( nn = 0; nn <= 30; ++nn )
    out[nn] ^= 0x40u;
  for ( i1 = 0; i1 <= 30; ++i1 )
    out[i1] ^= 0x57u;
  for ( i2 = 0; i2 <= 30; ++i2 )
    out[i2] ^= 0x2Eu;
  for ( i3 = 0; i3 <= 30; ++i3 )
    out[i3] ^= 0x49u;
  for ( i4 = 0; i4 <= 30; ++i4 )
    out[i4] ^= 0x37u;
  byte_40807F = 0;
  if ( !strcmp(out, aim) )
    result = puts("Congratulations!");
  else
    result = puts("no...Don't Give up!");
  return result;
}
```

可以看到, 它是将输入依次对 0x17 0x39 0x4A ...... 0x37 进行异或, 然后将运算结果out与aim进行比较. 双击aim变量名, 跳转到栈帧中, 得到aim的值```rpz|kydKw^qTl@Y/m2f/J-@o^k.,qkb```.

由 ```a^b^b == a```, 不难发现将aim依次同 0x37 0x49 ...... 0x39 0x17 进行异或即可得到flag```moectf{ThAnKs_F0r-y0U2_pAt13nt}```

~~是挺需要耐心的...~~

### Protection

下载文件, 根据提示, 程序应该是加了个壳... 检测一下, 是一个UPX的壳, 根据 [UPX的文档](https://linux.die.net/man/1/upx), 我们可以使用```-d```选项解压缩.

![image-20201017215239958](MoeCTF%202020%20WriteUp/image-20201017215239958.png)

去掉UPX壳之后, 又到了IDA大显神通的时间了. 我们得到main函数的伪代码如下.

``` c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  signed int i; // [rsp+Ch] [rbp-34h]
  char v5[40]; // [rsp+10h] [rbp-30h]
  unsigned __int64 v6; // [rsp+38h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  printf((unsigned __int64)"please input your flag: ");
  _isoc99_scanf((unsigned __int64)"%28s");
  for ( i = 0; i <= 27; ++i )
  {
    if ( ((unsigned __int8)x[i] ^ (unsigned __int8)v5[i]) != y[i] )
    {
      puts("wrong!", v5);
      return 0;
    }
  }
  puts("right!", v5);
  return 0;
}
```

不难看出, 这里是将输入和 x 进行逐个字符异或后再同 y 比较. 用同样的方法, 我们得到```x = aouv#@!V08asdozpnma&*#%!$^&*```, `y ={0x0c, 0x0, 0x10, 0x15, 0x57, 0x26, 0x5a, 0x23, 0x40, 0x40, 0x3e, 0x42, 0x37, 0x30, 0x9, 0x19, 0x3, 0x1d, 0x50, 0x43, 0x7, 0x57, 0x15, 0x7e, 0x51, 0x6d, 0x43, 0x57, 0, 0, 0, 0}`. 

写个Java脚本 ~~暂时还不会py~~ 异或之后得到 flag `moectf{upx_1S_simp1e-t0_u3e}`

~~UPX是挺好用的~~

## Algorithm

算法题, 据说在正式的CTF比赛中不会出现...

### mess

~~What a mess !~~

查看Python脚本, 发现是将flag转为ASCII码之后, 再向其中随机插入字母.

解法也简单, 去掉字母后将相邻的两个(或三个数字, 范围是 20~126 )转换为该ASCII码对应的字符.

flag: `moectf{pyth0n_1s_s0_s1mple}`

### 曲奇饼

统计非重复子串的最大长度.

简单的滑动窗口即可解决, 细节参见代码.

``` c
#include<cstdio>
#include<iostream>

using namespace std;

int getMax(int a, int b) { //或者用自带的max也行
    int max = a;
    if (b > max) {
        return b;
    } else {
        return a;
    }
}

int getMaxSubStrLeng(string inStr) {
    //滑动窗口, 应该很直观
    int max = 0, currBegin = 0, currEnd = 0;
    int len = inStr.size();
    string currContent = "";
    while (currBegin < len && currEnd < len) {
        if (currContent.find(inStr[currEnd]) == currContent.npos) {
            currContent.insert(currContent.end(), inStr[currEnd]);
            currEnd++;
            max = getMax(max, currEnd - currBegin);  //更新最大值
        } else {
            //cout << currContent.find(inStr[currBegin]) << endl;
            currContent.erase(currContent.find(inStr[currBegin]), 1);
            currBegin++;
            //遇到重复则逐渐从左端删除currContent中的字符,
            //直到不再有字符和inStr[currEnd]重复
        }
    }
    //cout << currContent << endl;
    return max;
}

int main(void) {
    string inStr;
    getline(cin, inStr);
    cout << getMaxSubStrLeng(inStr) << endl;
}
```



### Frank & 赤道企鹅, 永远的神

这两道题解法相似, 都是统计文件内容, 可参见repo中对应的FrankCounter.java 与EquatorCounter.java

其余略😜



## Classic Crypto

> "古典密码在现代的CTF比赛中已经很少出现"    -- FAQ

### 大帝的征程 #1 & #2 

凯撒密码是一种经典的移位密码, 由flag格式, 得前六个字符为`moectf`, 然后在此基础上计算密文相对于已知明文的偏移量即可

~~已知明文攻击 (大概)~~

### 大帝的征程 #3

由上述经验, 我们推测偏移量应该为 +47(注意由于ASCII码的范围为0~127, 我们需要对127取余作为结果), 写个java脚本解密, 我们得到了`moectf{cnquer_th_X#S$"}`...

看起来大部分是正确的, 只是有一些字符没有正确的解出来. 结合之前的经验, 我们推测`cnquer` 应该是`c0nquer`, `th`应该是`th3`. 计算可得这时的偏移量是-47, 再计算一次, 我们得到了全部明文: flag `moectf{c0nquer_th3_XDSEC}`

~~I'm afraid I'm too vegetable to do that 😭~~

### 大帝的征程 维吉尼亚 & 维吉尼亚Ex

#### 背景知识: 维吉尼亚密码

维吉尼亚密码由凯撒密码扩展而来，引入了密钥的概念。即根据密钥来决定用哪一行的密表(或者你也可以理解成偏移量...)来进行替换，以此来对抗字频统计。

也就是说, 实际上, 维吉尼亚分解之后还是相当于多个凯撒密码的组合...

那现在(至少相邻字符的)偏移量不再固定, 我们应该怎么做呢...?



#### Friedman测试法确定密钥长度

我们知道, 不同字母在同一文本的出现概率并不相同(至少对于包括英文在内的大多数语言如此). 那么, 我们可以定义一个叫做"重合指数"的量, 来量化两个串之间的对应程度.

设x=x1x2...xn是一条n个字母的串，x的重合指数记为CI，定义为x中两个随机元素相同的概率。

而通过对大量的英文文本进行统计, 我们得到每个字母的出现概率如下. 

![【密码学】维吉尼亚密码加解密原理及其破解算法Java实现](MoeCTF%202020%20WriteUp/b91e4e49c93c2db554922f8ecb22f56a_thumb.png)

由此, 我们可以得到英文文本的重合指数为 **0.065**(我们假定明文的重合指数大致也是这个值).

结合之前的"维吉尼亚密码相当于多个凯撒密码的组合"这一特点, 我们就可以把长为n的密文先分为m组, 按照列的形式组成一个 m*(n/m)的矩阵, 然后检测每一组密文的重合指数. 

如果m正好就是密钥的长度, 那么每一组的CI值应当大致也是0.065; 如果不是的话, 那么分组后的密文看起来应该很随机. 而对于一个随机串, 重合指数约为 **0.038**.

综上, 我们就通过Friedman测试法得到了密钥的最可能长度.



#### 字母频度分析确定密钥

有了密钥长度m, 我们就可以将密文按照密钥长度分解为k = n/m组, 每一组都是凯撒密码, 然后用类似上图的字母频度分析即可解密.

> 由上面的分析可知, 要破解维吉尼亚密码, 还是需要收集到大量的密文的. 不过这两道题的密文都很长, 足够破解了.



## Android

其实Android我之前也没接触过, 这里就简单的记录一下简单的Android逆向过程吧.

 ~~"你之前到底接触过啥??" "啥也没接触过...."~~

![img](MoeCTF%202020%20WriteUp/5442499-52cadc4569714633.png)

借一张图(来自[这里](https://www.jianshu.com/p/d29c37dda256))说明apk的打包过程.

我们看到, java文件在编译后会生成class文件, 然后与其他第三方的class文件与library一起组成dex文件, 然后再经过一些处理, zip压缩, 签名, 对齐后就是我们看到的apk文件了.

所以, 我们可以直接将apk按照zip解压, 然后获得dex文件. 不过dex是Android的Dalvik虚拟机专用的字节码文件, 我们得把它转成标准的java虚拟机的class文件. 

幸运的是, 已经有大佬实现了相关的工具: [dex2jar](https://github.com/pxb1988/dex2jar). 获得jar/class文件后, 我们可以使用[jd-gui](https://github.com/java-decompiler/jd-gui)反编译class文件, 得到源码.



### Baby Android

(稍后再更...)
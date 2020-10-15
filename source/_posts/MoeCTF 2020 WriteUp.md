# MoeCTF 2020 Write Up

***By Horizon Chaser, aka. Horizon***

第一次参加CTF, 现学现用, 会做的题实在有限😂, 先把自己会的或者有思路的部分题写下来, 以供参考

~~其他不会的题就等各位巨佬的Write Up辽~~

## Misc

Misc, 全程Miscellaneous, 本意是"杂项". 在CTF中大概是指多个领域 ~~脑洞~~ 的混合. 因此做起来还是很有意思的.

### Welcome

Misc入门题, 附件profession.jpg. 打开, 发现是专业团队, 仔细一看右下角有黑白色块, 觉得可能是在jpg文件末端添加了内容.

使用binwalk分析, 未发现隐写的文件, 有点疑惑.

使用16进制编辑器打开, 发现flag就在末尾😂

![image-20201012183142831](MoeCTF%202020%20WriteUp.assets/image-20201012183142831.png)

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

``` C
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






























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

### SimpleRev

DIE 检查, 发现是一个 x64 ELF 文件. IDA 打开, 定位到 `main`.

```C
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  int v3; // eax
  char v4; // [rsp+Fh] [rbp-1h]

  while ( 1 )
  {
    while ( 1 )
    {
      printf("Welcome to CTF game!\nPlease input d/D to start or input q/Q to quit this program: ");
      v4 = getchar();
      if ( v4 != 'd' && v4 != 'D' )
        break;
      Decry();
    }
    if ( v4 == 'q' || v4 == 'Q' )
      Exit("Welcome to CTF game!\nPlease input d/D to start or input q/Q to quit this program: ", argv);
    puts("Input fault format!");
    v3 = getchar();
    putchar(v3);
  }
}
```

明显, 在 `Decry` 函数中进行了 flag 的验证....

```C
unsigned __int64 Decry()
{
  char currChar; // [rsp+Fh] [rbp-51h]
  int v2; // [rsp+10h] [rbp-50h]
  int v3; // [rsp+14h] [rbp-4Ch]
  int i; // [rsp+18h] [rbp-48h]
  int keyLen; // [rsp+1Ch] [rbp-44h]
  char src[8]; // [rsp+20h] [rbp-40h] BYREF
  __int64 v7; // [rsp+28h] [rbp-38h]
  int v8; // [rsp+30h] [rbp-30h]
  char *v9; // [rsp+40h] [rbp-20h] BYREF
  __int64 v10; // [rsp+48h] [rbp-18h]
  int v11; // [rsp+50h] [rbp-10h]
  unsigned __int64 v12; // [rsp+58h] [rbp-8h]

  v12 = __readfsqword(0x28u);
  *(_QWORD *)src = 'SLCDN';
  v7 = 0LL;
  v8 = 0;
  v9 = (char *)'wodah';
  v10 = 0LL;
  v11 = 0;
  text = join(key3, (const char *)&v9);         // text == killshadow
  strcpy(key, key1);                            // key == ASDFK
  strcat(key, src);                             // key == ASDFKNDCLS
  v2 = 0;
  v3 = 0;
  getchar();
  keyLen = strlen(key);
  for ( i = 0; i < keyLen; ++i )
  {
    if ( key[v3 % keyLen] > '@' && key[v3 % keyLen] <= 'Z' )
      key[i] = key[v3 % keyLen] + 32;
    ++v3;
  }
  printf("Please input your flag:");
  while ( 1 )
  {
    currChar = getchar();
    if ( currChar == '\n' )
      break;
    if ( currChar == ' ' )
    {
      ++v2;
    }
    else
    {
      if ( currChar <= 96 || currChar > 122 )   // NOT lower case
      {
        if ( currChar > '@' && currChar <= 'Z' )// upper case
        {
          str2[v2] = (currChar - 39 - key[v3 % keyLen] + 97) % 26 + 97;
          ++v3;
        }
      }
      else
      {
        str2[v2] = (currChar - 39 - key[v3 % keyLen] + 97) % 26 + 97;
        ++v3;
      }
      if ( !(v3 % keyLen) )
        putchar(' ');
      ++v2;
    }
  }
  if ( !strcmp(text, str2) )
    puts("Congratulation!\n");
  else
    puts("Try again!\n");
  return __readfsqword(0x28u) ^ v12;
}
```

在 37 行的 `while` 循环后, 通过计算 `str2[]`的值并判断是否与`text`相等来确定 flag 正确与否, 写个脚本爆破即可.

```python
upperTable='ABCDEFGHIJKLMNOPQRSTUVWXYZ'
lowerTable = upperTable.lower()
key=list('ADSFKNDCLS'.lower())
klens=len(key)

text='killshadow'
flag=''
flag2=''
for i in range(len(text)):
    str2=text[i]
    for c in upperTable:
        if str2== chr((ord(c) - 39 - ord(key[i  % klens]) + 97) % 26 + 97):
            flag+=c

for i in range(len(text)):
    str2=text[i]
    for c in lowerTable:
        if str2== chr((ord(c) - 39 - ord(key[i  % klens]) + 97) % 26 + 97):
            flag2+=c
print('flag{'+flag+'}')
print('flag{'+flag2+'}')
```

因为并未对大小写进行限定, 所以大小写理论上均可...大概, 总之大写是可以的.

`flag{KLDQCUDFZO}`

## Week 6, 02/07 - 02/14, BUUCTF

### CrackRTF

看题目说明, 在文件中藏了一个 rtf 文件, 不过用 binwalk 扫描没发现什么. 可能是加密了...

用 IDA 打开, 定位到 `main_0`, 发现需要两个密码, 长度均为 6 个字符, 其中第一个为纯数字, 与 `@DBApp` 链接后计算一个哈希值, 与 `6E32D0943418C2C33385BC35A1470250DD8923A9` 比较.

```C
int __cdecl main_0(int argc, const char **argv, const char **envp)
{
  DWORD v3; // eax
  DWORD v4; // eax
  char Str[260]; // [esp+4Ch] [ebp-310h] BYREF
  int v7; // [esp+150h] [ebp-20Ch]
  char String1[260]; // [esp+154h] [ebp-208h] BYREF
  char Destination[260]; // [esp+258h] [ebp-104h] BYREF

  memset(Destination, 0, sizeof(Destination));
  memset(String1, 0, sizeof(String1));
  v7 = 0;
  printf("pls input the first passwd(1): ");
  scanf("%s", Destination);
  if ( strlen(Destination) != 6 )
  {
    printf("Must be 6 characters!\n");
    ExitProcess(0);
  }
  v7 = atoi(Destination);
  if ( v7 < 100000 )
    ExitProcess(0);
  strcat(Destination, "@DBApp");
  v3 = strlen(Destination);
  getSHA1((BYTE *)Destination, v3, String1);
  if ( !_strcmpi(String1, "6E32D0943418C2C33385BC35A1470250DD8923A9") )
  {
    printf("continue...\n\n");                  // Destination == 123321@DBApp
    printf("pls input the first passwd(2): ");
    memset(Str, 0, sizeof(Str));
    scanf("%s", Str);
    if ( strlen(Str) != 6 )
    {
      printf("Must be 6 characters!\n");
      ExitProcess(0);
    }
    strcat(Str, Destination);
    memset(String1, 0, sizeof(String1));
    v4 = strlen(Str);
    getMD5((BYTE *)Str, v4, String1);
    if ( !_strcmpi("27019e688a4e62a649fd99cadaafdb4e", String1) )
    {
      if ( !(unsigned __int8)sub_40100F(Str) )
      {
        printf("Error!!\n");
        ExitProcess(0);
      }
      printf("bye ~~\n");
    }
  }
  return 0;
}
```



进入 `sub_401230`, 定位到 `CryptCreateHash(phProv, 0x8004u, 0, 0, &phHash)`, 搜一下这个函数的[文档](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id), 发现第二个参数控制了计算的哈希种类, `0x8004`是 SHA1, `0x8003`是 MD5. 由此我们可以对在 [100000, 1000000) 的前半部分进行爆破, 写个脚本.

```python
import hashlib

prefix = ""
postfix = "@DBApp"

for i in range(100000, 1000000):
    prefix = str(i)
    shaObj = hashlib.sha1(bytes(prefix + postfix, encoding="UTF-8"))
    # print(shaObj.hexdigest())
    if(shaObj.hexdigest() == "6E32D0943418C2C33385BC35A1470250DD8923A9".lower()):
        print(prefix + postfix)
     
```

这样, 我们得到了第一个密码 `1233321`, 这时 `Destination == "123321@DBApp"`, 输入的第二个密码后面会连接 `Destination`, 然后计算 MD5 值, 和 `27019e688a4e62a649fd99cadaafdb4e`比较, 判断是否正确.

需要注意的是, 第二个密码只限制了长度, 没有限制是纯数字...~~没注意到的话你可能就会和我一样卡住了~~

到这里, 我们有两种方法.

#### 充分利用你电脑的计算能力

我们已知第二个 MD5 原文的后半部分和前半部分的长度, 相当于一个加盐的 MD5 爆破 - [HashCat ](https://github.com/hashcat/hashcat)很适合干这个事儿, 特别是你的 GPU 比较强的时候.

用如下的命令行进行一次 `md5($pass.$salt)`掩码攻击. ~~其实就是暴力穷举, 不过更高级一点, 大概~~

`.\hashcat.exe -m 10  "27019e688a4e62a649fd99cadaafdb4e:123321@DBApp" -a 3 ?a?a?a?a?a?a`

在我的电脑上, 大概十秒就得到了结果...

![image-20210207140651360](2021-Weekly-Reverse/image-20210207140651360.png)

前半部分是 `~!3a@0`, 把俩密码输入程序, 得到一个 RTF 文档, 打开就是 flag.

~~不过, 我认为这应该不是出题人的最初想法吧~~



#### 充分利用你自己的 ~~计算能力~~ 解题能力

在 `main_0` 的 44 行, 我们发现在两个密码验证正确后, 调用了 `sub_40400F`, 参数是 `Str`. 

`sub_40400F`会再调用 `sub_4014D0`, 参数变为 `LPCSTR`类型.

```c
char __cdecl sub_4014D0(LPCSTR lpString)
{
  LPCVOID lpBuffer; // [esp+50h] [ebp-1Ch]
  DWORD NumberOfBytesWritten; // [esp+58h] [ebp-14h] BYREF
  DWORD nNumberOfBytesToWrite; // [esp+5Ch] [ebp-10h]
  HGLOBAL hResData; // [esp+60h] [ebp-Ch]
  HRSRC hResInfo; // [esp+64h] [ebp-8h]
  HANDLE hFile; // [esp+68h] [ebp-4h]

  hFile = 0;
  hResData = 0;
  nNumberOfBytesToWrite = 0;
  NumberOfBytesWritten = 0;
  hResInfo = FindResourceA(0, (LPCSTR)0x65, "AAA");
  if ( !hResInfo )
    return 0;
  nNumberOfBytesToWrite = SizeofResource(0, hResInfo);
  hResData = LoadResource(0, hResInfo);
  if ( !hResData )
    return 0;
  lpBuffer = LockResource(hResData);
  sub_401005(lpString, (int)lpBuffer, nNumberOfBytesToWrite);
  hFile = CreateFileA("dbapp.rtf", 0x10000000u, 0, 0, 2u, 0x80u, 0);
  if ( hFile == (HANDLE)-1 )
    return 0;
  if ( !WriteFile(hFile, lpBuffer, nNumberOfBytesToWrite, &NumberOfBytesWritten, 0) )
    return 0;
  CloseHandle(hFile);
  return 1;
}
```

首先是 `FindResourceA`这个函数, 从[文档](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-findresourcea)来看, 它会尝试寻找名称为 `0x65`, 类型为 `AAA` 的资源, 然后通过`SizeofResource`计算长度, 通过`LoadResource`加载到`lpBuffer`中并上锁, 之后调用了`sub_401005`进行了什么操作, 然后创建`dbapp.rtf`并写文件, 最后关闭.

看来`sub_401005`就是解密函数了.

`sub_401005`会继续调用`sub_401420`, 主要是一个异或的循环...

```C
unsigned int __cdecl sub_401420(LPCSTR key, int fileContentOffset, int a3)
{
  unsigned int result; // eax
  unsigned int i; // [esp+4Ch] [ebp-Ch]
  unsigned int keyLen; // [esp+54h] [ebp-4h]

  keyLen = lstrlenA(key);
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i >= a3 )
      break;
    *(_BYTE *)(i + fileContentOffset) ^= key[i % keyLen];
  }
  return result;
}
```

在这里, 会将现有数据的每一位和密钥进行异或作为解密, 而原文件是一个 RTF 格式的文件, 文件头是 [`7B 5C 72 74 66 31`](https://www.filesignatures.net/index.php?page=search&search=7B5C72746631&mode=SIG), 正好六个字节, 和第二段密码一样长.

参考[这篇题解](https://blog.csdn.net/qq_43786458/article/details/102488408), 用 ResourceHacker 看一下资源,果然找到了这个. (0x65 == 101)

![image-20210207150448312](2021-Weekly-Reverse/image-20210207150448312.png)

加密后的文件头是 `05 7D 41 15 26 01 `, 把它和原来的文件头进行异或就得到了第二段密码: `~!3a@0`

`flag{N0_M0re_Free_Bugs}`


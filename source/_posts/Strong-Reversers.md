---
title: Strong-Reversers
date: 2021-01-27 14:39:10
tags:
- Reverse
- Android
---

## 0x00 DDCTF-Android Easy

~~接触到的第二道安卓逆向题~~

下载, 发现是个 zip , 但是有 apk 的目录结构, 用 jadx-gui 打开可以看到如下的结构

![image-20210127124159278](https://i.loli.net/2021/01/27/Jd8H6ITzRl5NYsO.png)

很明显, 重点在 FlagActivity 类里面, `i()`中将`p` `q` 两个byte数组进行一系列操作后得到解密后的`byte[] bArr2`, 作为`String()`的参数返回.  之后在`onClickTest()`中通过将输入的字符串同`i()`的返回值进行比较, 判定 flag 是否正确.

那就很显然, `i()`的返回值就是正确的 flag. 把`i()`运行一次, 得到返回值`DDCTF-3ad60811d87c4a2dba0ef651b2d93476@didichuxing.com`, 用`flag{ }`包裹提交.

`flag{DDCTF-3ad60811d87c4a2dba0ef651b2d93476@didichuxing.com}`



## 0x01 WELCOME TO JNI

> "JNI是什么？"

> **JNI** （**Java Native Interface，Java本地接口**）是一种[编程框架](https://zh.wikipedia.org/w/index.php?title=编程框架&action=edit&redlink=1)，使得[Java虚拟机](https://zh.wikipedia.org/wiki/Java虚拟机)中的[Java](https://zh.wikipedia.org/wiki/Java)程序可以调用本地应用/或库，也可以被其他程序调用。 本地程序一般是用其它语言（[C](https://zh.wikipedia.org/wiki/C语言)、[C++](https://zh.wikipedia.org/wiki/C%2B%2B)或[汇编语言](https://zh.wikipedia.org/wiki/汇编语言)等）编写的，并且被编译为基于本机硬件和操作系统的程序。
>
>
> 
> -- Wikipedia

简单的来说, JNI 可以让 Java 调用其他语言的库.

用 jadx-gui 打开 apk 文件, 定位到 Main Activity --在`com.reverier.xdsec_re_20200126`下面

![image-20210127131115411](https://i.loli.net/2021/01/27/hwg4BSpNfiFO8DH.png)

在`MainActivity`类中, 可以看到声明了一个 native 方法 - `loginUtils()`, 从名字推测是 ~~检查 flag~~ 登陆验证, 加载了一个本地库`native-lib`, 它对应的文件在`/lib`下面, 对应不同的架构.

在 33 行可以看到, `loginUtil()`接受了输入的字符串作为参数, 然后返回一个布尔值作为结果, 控制输出`Right`和`Wrong` - 这就是重点了.

从 apk 中提取出 x86 架构对应的`native-lib.so`, IDA 打开, 找到对应的方法`Java_com_reverier_xdsec_1re_120200126_MainActivity_loginUtils()`, 反编译如下.

![image-20210127132734025](https://i.loli.net/2021/01/27/7UM1xOPLB3RTIFK.png)

第 8 行开始, `v6`保存了作为参数的字符串的长度, `v5`则保存了另一个字符串的长度, `v4`保存了参数字符串. 第 11 行比较两个字符串的长度, 若相等则再通过`strncmp()`比较. 

综上, `off_1FD4 + 5972`应该就指向了flag. `0x1FD4 + 5972d == 0x880`, 跳转过去, 发现果然保存着flag.

![image-20210127133107771](https://i.loli.net/2021/01/27/shcxbOGXJ2U6DFV.png)

`flag{welcome_to_naive_lib!}`

做完了才意识到, 其实当时直接从 IDA 的 Strings window 能直接看到这个明文字符串...



## 0×02 Codegate CTF 2018 RedVelvet

IDA 打开, 跳转到`main()`, 发现了一大串`funcX()`的调用.  ~~有点壮观(x~~

观察结构发现, 在 48 行, ``fgets()`接受了 28 个字节的输入(包含末尾的`\n`), 保存到`s`中. 而`funcX()`并未改变`s`的值, 而是进行了一些验证, 比如`func7()`: 

![image-20210127134648665](https://i.loli.net/2021/01/27/fuRYoAhm31TjLzw.png)

这 15 个`funcX()`共同对`s`进行了一系列的检查, 然后计算`s`的 SHA256 值, 并和`0a435f46288bb5a764d13fca6c901d3750cee73fd7689ce79ef6dc0ff8f380e5`比较, 确定 flag 正确与否.

~~所以直接用 hashcat 穷举破解理论上倒也可行~~

接下来就是 angr 发挥威力的时候了, 我们不需要将程序执行完, 只需要找到一个输入, 能够满足这十五个`funcX()`的约束, 使程序运行到`SHA256_Init()`前即可 - 对应的地址是`0x401534`.

同时, 我们还需要避免进入`funcX()`中的`exit(1)`的分支, 以`func1()`为例.

![image-20210127135526675](https://i.loli.net/2021/01/27/Mu2QcKWZ81dfzFR.png)

`0x4009ED`和`0x4009F7`就是我们不希望运行到的地方, 因为到这里说明我们的输入没有通过`func1()`的检验, 执行了`exit(1)` - 其他的`funcX()`同理.

这样, 我们得到了期望执行到的地址与要避免的地址, 写出如下脚本.

```python
import angr

prog = angr.Project('./RedVelvet', load_options={'auto_load_libs': False})   
state = prog.factory.entry_state()    
simgr = prog.factory.simgr(state)   

simgr.explore(find=0x00401534 ,avoid=[0x4009ED,0x4009F7,0x400A3C,0x400A46,0x400A9F,0x400B01,0x400B5C,0x400C05,0x400CAB,0x400D51,0x400DD6,0x400E5E,0x400F07,0x400FAD,0x4105F,0x4010E9, 0x40119D]) 

flag = simgr.found[0].posix.dumps(0)
print(flag)


```

经过漫长的运行( VMWare Ubuntu + Docker angr/angr 大概 30 分钟? ), 我们得到了如下输出 (`fg`是因为我之前误以为写错了, 于是挂起去检查脚本了...)

![image-20210127140006667](https://i.loli.net/2021/01/27/id16RHAoK5TBpW3.png)

放到源程序里检查一下, 看来没毛病. 

![image-20210127140226953](https://i.loli.net/2021/01/27/DeuYsaXi3ZW4mMt.png)

`flag{What_You_Wanna_Be?:)_la_la}`

### Something Else

1. RedVelvet依赖 1.0.0 版本的 libcrypto.so, 但是包含它的老版本的 openssl 已经过时了, 最后用`apt-file`查到英伟达的`nslight-system`还带这东西, 于是安装之后手动复制出来...
2. 理论上通过 15 个`funcX()`中的约束条件, 可以直接求出来满足的输入值, 就像`z3`那样
3. 如果限定输入长度与范围( ASCII 可见字符) 的话, 应当能够跑的更快, 学习中
4. 关于原题: 暂时没找到....
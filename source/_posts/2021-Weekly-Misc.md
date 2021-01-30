---
title: 2021 Weekly Misc
date: 2021-01-29 15:17:16
excerpt: 每周的Misc练习与Writeup
tags:
- Misc
---

##  Week 4, 01/24 - 01/30 BUUCTF

### 被偷走的文件

打开流量包, 发现一些FTP协议的包, 用`ftp or ftp-data`筛选出来, 结果如下.

![image-20210130110430119](2021-Weekly-Misc/image-20210130110430119.png)

55 号包就是服务器返回的包含 flag.rar 的数据包, 右键详情中的 FTP Data 分组, 导出分组字节流. 从文件头的`Rar!`可以判定, 这是一个 rar 文件.

打开导出的文件, 发现需要密码, 用 ARCHPR 爆破, 得到四位密码`5790`, 进而得到 flag.

`flag{6fe99a5d03fb01f833ec3caa80358fa3}`

#### 另一种提取数据的做法

从 [pcapng 的规范](https://pcapng.github.io/pcapng/draft-tuexen-opsawg-pcapng.html)中, 我们可以发现数据包在捕获文件中是明文存储的, 同时 FTP 的数据也是明文传输的, 所以我们可以直接使用 binwalk 或者 foremost 从 pcapng 中提取文件.

如果提取出的文件太多的话, 也可以先导出指定的数据包再尝试提取.



### [BJDCTF2020] 认真你就输了

打开, 是一个 xls 文件, 但不能正常显示.

不过 xls 文件本身也是一个 zip 压缩包, 直接解压, 在 xl/charts 下面发现一个 flag.txt, 打开就是 flag...

`flag{M9eVfi2Pcs#}`



### [BJDCTF2020]藏藏藏

打开, 是张 jpg 图片, 用 stegsolve 看一下 File Format...

![image-20210130121745206](2021-Weekly-Misc/image-20210130121745206.png)

看来文件末尾藏了点东西, 熟悉的 `50 4B 03 04` - 应该是个 zip . 比较奇怪的是, 用 binwalk 没能识别到 zip 头, 只识别到了结尾...

![image-20210130122312148](2021-Weekly-Misc/image-20210130122312148.png)

不过关系不大, 用 UltraEdit (或者别的什么十六进制编辑器)打开, 定位到 zip 文件头, 在`0xC7EE`的位置, 写个 python 脚本从这里切分就行.

打开切分得到的 zip 文件, 里面是个 docx 文件, 打开扫码, 得到flag.

`flag{you are the best!}`



### [GXYCTF2019] 佛系青年

打开压缩包, 里面有一张图, 一个 txt, 后者是加密的.

![image-20210130151231685](image-20210130151231685.png)

到这里, 会产生两种想法: 

- 压缩包的密码在图里, 得先把密码找到 (错了)
- **压缩包是伪加密的** (这才是对的...)

#### 错误示范

用 stegsolve 尝试无果, 尝试用 zsteg 检测隐写方式, 发现不支持 - 文件头是 `FF D8 FF D9`...原来是个 jpg  ~~又被出题人套路了,下次一定注意先用file确认一下~~

用 stegdetect 检测, 系数为 10.0, 报告可能是 jphide, 用 stegbreak 跑字典, 未果.

至此发现, 此路不通...😭

#### 正解

因为存在未加密的文件, 因此一定不是全局伪加密, 可能是单独设置了每一个文件记录的加密位.

用 010 Editor 打开, 定位到 dirEntry[1], 也就是 fo.txt 对应的记录的位置, 发现 [`deFlags`](https://www.jianshu.com/p/8e4209bca4af)是`09 00`, 存在伪加密, 修改为`00 00`, 正常解压出了 fo.txt.

![image-20210130152220026](image-20210130152220026.png)

在最下面我们发现了这么一串东西.

> 佛曰：遮等諳勝能礙皤藐哆娑梵迦侄羅哆迦梵者梵楞蘇涅侄室實真缽朋能。奢怛俱道怯都諳怖梵尼怯一罰心缽謹缽薩苦奢夢怯帝梵遠朋陀諳陀穆諳所呐知涅侄以薩怯想夷奢醯數羅怯諸

在[这里](http://www.keyfc.net/bbs/tools/tudoucode.aspx)解密, 得到 flag.

`flag{w0_fo_ci_Be1}`

~~所以那张图真的没有用啊....~~

### 秘密文件

打开流量包, 发现大量 FTP 协议流量, 跟踪.

![image-20210130153937974](image-20210130153937974.png)

发现一个 rar 文件, binwalk 分离, 发现有密码, 爆破, 得口令为`1903`, 解压得 flag.

`flag{d72e5a671aa50fa5f400e5d10eedeaa5}`

~~我一开始还以为密码是ctf来着...~~
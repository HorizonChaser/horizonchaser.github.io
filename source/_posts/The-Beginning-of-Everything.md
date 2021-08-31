---
title: About Horizon Blog
date: 2020-10-15 21:52:08
tags: 
- 随笔
- Develop
---

## The Beginning of Everything

其实很早之前就想在 Github Pages 上搭一个个人博客了, CSDN 总是让人很难受

 ~~但是拖延症晚期的我现在才搭完hhh~~

言归正传, 基于 Hexo 和 Travis CI 的 Pages 并不难部署, 只是基于 YMAL 的主题配置写起来有点麻烦, 向 Markdown 中插入图片也是一件麻烦事儿....

博客在 [Gitee Pages](https://horizonchaser.gitee.io/) 上有一份镜像, ~~不过在我搞定 Github 的 workflow 之前, Gitee 的更新可能不是很及时😂~~已经搞定了🍻

就这样吧, 欢迎大家来玩儿~

## Known Issue

- [ ] Gitalk 由于 Github 的 CORS 限制无法使用

## Develop Plan

开发计划 ~~挖坑计划~~

- [ ] 添加一个 Live2D 看板娘
- [ ] 魔改一下 `hexo-bilibili-bangumi`, 加个进度显示
- [ ] 加个音乐播放器
- [ ] 把 RSS 订阅显示的更新日期改成文章的更新日期
- [ ] 整个随机 Banner

## 更新日志

### Update 2020/10/15

Horizon Blog 搭起来了! 👏👏👏

### Update 2021/01/31

把博客的 CI 从 Travis 迁移到 Github Actions 上了, 前者太慢了, 排队能排半个小时...

### Update 2021/02/05

从 [蝴蝶计数器](https://www.bfcounter.vip/) 拿到了一个统计的小工具, 放到这里



~~看起来并没有多少人看...~~

### Update 2021/06/14

把 hexo 升级到了 hexo 5, 主题和其他依赖包也一并升级了下

为了~~更好看地~~用 LaTex 写公式, 把渲染器换成了 `hexo-renderer-markdown-it-plus`, 并用 `katex` 作为解析器

### Update 2021/06/15

增加了 RSS 订阅支持

### Update 2021/08/31

修好了 RSS 订阅地址, 原来一直是 [example.com](example.com), 还在想为啥, 原来是因为 `_config.yml` 里面的地址忘改了...

---
title: 使用 GitHub Action 自动推送博客更新到 Telegram 频道
date: 2024-01-09 23:39:32
excerpt: 利用 Action, 我们不仅可以自动化构建与发布博客, 还可以推送更新消息
tags:
- GitHub
- CI/CD
- Telegram Bot
- GitHub Action
---

# 使用 GitHub Action 自动推送博客更新到 Telegram 频道

之前在别人的频道看到了将自己博客更新的消息推送到自己频道的 Bot, 觉得很不错, 所以 --- 咱也要整一个!

折腾了一小段时间之后, 就有了这篇文章, 以及一个推送的 Bot.

## 整体流程

这个事情说起来也不麻烦, 博客本身就是使用 Hexo 通过 GitHub Action, 在每次 push 时自动构建和发布的. 所以我们要做的事情就是, 在 Action 流程中加一个步骤, 获取这次提交包含的 Markdown 文件, 然后通过 Telegram Bot API 发送到自己的频道:

1. 获取当前的 Commit Message, 判断是否包含 `upd post` 或 `new post`
2. 如果包含, 则通过 `gitpython` 库获得当前提交包含的 Markdown 文件, 再通过 head matter 的第二行拿到名称, 第三行拿到摘要
3. 拼一下消息字符串, 通过 `telegram` 库, 用机器人账号把消息发到指定的位置
   1. Bot 通过 BotFather 创建, **记得手动加到频道里并给对应的发送消息的权限**
   2. Bot 的 Token 和目标频道的 ID 放到 Secret 里面, 通过环境变量在执行时获取
      1. 虽然 Action 会试着给你包含这些内容的输出 (如果真有的话) 打码, 但你要是直接写在源码里仓库还是 public 的话谁也救不了

整体的流程就是这样, 并不算难, 麻烦的地方在于获取具体哪篇是更新的文章 --- Hexo 本身似乎没有提供 API, GPT 给出的脚本也跑不了, 大概是瞎编的. 而折腾这么久的原因主要是两个

1. 没整明白如何在 Action 的不同步骤之间传递消息, 不管是 `echo ::setoutput` 还是写到环境文件都没效果
2. `telegram` 库中发送消息的方法是 `async` 的, 而某个笨蛋很长时间没写过这类代码也没看文档

## 待改进之处

还是有很多要改进的地方的:

1. 更新信息里面附带摘要 (是的上面写了但是我没真正加进去)
2. 把上面的第一步和第二步都放到 py 脚本里面去, 由于历史遗留问题这两步现在是分开的

等我有时间再改改吧, 最近还是比较忙

就这样, 博客的 Action 配置文件在[这里](https://github.com/HorizonChaser/horizonchaser.github.io/blob/backup/.github/workflows/main.yml), 需要的话可以看看.

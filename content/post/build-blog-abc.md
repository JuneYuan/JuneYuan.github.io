---
title: "Blog 搭建过程"
date: 2018-09-02T09:06:29+08:00
draft: false
---

记录下自己的搭建过程和遇到的问题，不是教程，教程见下文 Reference.

## 要点

网上看了一些选型取舍，决定使用 Hugo + Travis CI + GitHub Pages 工具组合。三者都是第一次接触, 动手之前完全不知其然更不知其所以然，因此费了不少周折。

Hugo 是要安装的，所以首先搜索教程。homebrew 最方便，但安装以后发现，想使用的主题不支持 homebrew 默认版本，遂改为下载特定版本来安装。

然后是 GitHub Pages. 有个问题开始一直不明白：需要专门为 GitHub Pages 建一个 repo, 还是直接复用存放博客文本的 repo 就可以。结论是，Project Pages 不需要专门建 repo, 可以直接在 Project repo 的设置页面配置，而另一种 User/Organization Pages 似乎需要一个独立的 repo.

最后是持续集成工具 Travis CI 的配置和使用。目的是，每次执行 git push 后 Travis 都能自动完成拉取最新代码、生成静态网站、以及发布到 GitHub Pages 的整个流程。为了让 Travis 能成功访问仓库代码，需要给它配置一个 Personal Access Token.

## 踩坑

- Travis build 失败。一通 google 也未有所获，后来仔细看报错信息，发现需要将主题的 `exampleSite` 目录下的 `config.toml` 文件 copy 到自己站点根目录，而这一步没有做是因为跟着[官网快速搭建指南](https://gohugo.io/getting-started/quick-start/)走，config.toml 只写了极少的4行，并且添加 theme 时也是拿来即用，甚至忘了看人家的 README.

- localhost:1313 效果正常，但 github page 页面却显示不出主题的效果（看上去“全裸”），以及点击博客文章提示 404. 填坑花了几个小时 🙃 ，排查了各种可能：Travis 日志输出、主题版本、主题引入方式（起初是从 github 仓库 download 至本地，然后改为 [git submodule][1] 链接到作者的仓库）、检查页面源代码是否缺少 div 以及 css 等……最后找到问题，是因为 google 到别人用 Hexo 遇到[类似问题][2]，通过配置 `CNAME` 解决，这才受到启发，去检查自己 `config.toml` 文件的 `baseURL`, 发现没有加仓库名为后缀。

总结：

看官网教程很多时候是准确而高效的，步骤既清晰又不啰嗦，关键点会作必要提示，也给遇到问题 debug 提供了很有用的线索。follow 网上的博客教程，看似浅显易懂，走了捷径，实际效果则不然，尤其是中文博客鱼龙混杂，有时候作者自己对所写内容并不够懂，看这样的教程就可能被误导或带偏。

## Reference

- [博客写作工作流 | 这里有讨论博客选型取舍](https://blog.yuanbin.me/posts/2018-02/2018-02-23_23-19-29/)

- [使用Hugo搭建GitHub个人博客](https://www.jianshu.com/p/f1b02e00f206)

- [使用 Hugo 搭建博客](https://segmentfault.com/a/1190000012975914)

- [使用 Hugo _ Travis CI + Github Pages 搭建静态博客 | 这是看过几篇中写得最清晰的](https://devtian.me/post/crate-website-use-hugo/)


  [1]: https://stackoverflow.com/a/9357474/3762421
  [2]: http://m.codes51.com/itwd/4431697.html
---
title: "介绍一个搭建免费博客的实现方案"
date: 2023-12-06T15:01:02+08:00
draft: false
comment: true
description: "本文如何搭建一套免费的博客，实现可基于 markdown 撰写博客，使用免费环境部署，以及添加评论、图床和统计等附加能力。"
---

本文如何搭建一套免费的博客，实现可基于 markdown 撰写博客，使用免费环境部署，以及添加评论、图床和统计等附加能力。

这是一整套的搭建免费博客的可用方案。

## 前言概述

从几年前开始写博客，当时把博客建在一台云主机上。另外，还单独买了一个域名 - [poloxue.com](https://www.poloxue.com)。大概猜测，最初的大家都是有这种方式搭建自己博客。

这种方式有两点不令我满意：

首先是费用问题，虽说不是很贵，但一年也要 200-300，如果只是偶尔写写，不如发到一些内容平台。国内的博客平台还是挺多的，如 csnd、博客园、掘金等，都可利用。

再者是维护问题，单独购买一台云服务器，即使是偶尔可能的宕机，也是要维护查原因。我的博客在 2018 年搭建，后来不知道什么原因，突然宕机后，当时我工作太忙，停了好几年也没重开；

## 新的方案

互联网发展到今天，对博客这类常规需求，我们有免费的云服务使用，而且基本不会宕机，如果有大流量，也不用担心流量扩容的问题。

我的新方案就是，利用 Hugo + GitHub Page 免费搭建我的博客，实现一套免费的解决方案。其他组件如评论、图床和统计也都有免费的服务可用。

{{< image "2023-12/2023-12-06-create-your-own-free-blog-07.png" >}}

### Hugo

第一个要解决的问题是，如何如何编写博客？

我们可通过 markdown 编写文章，然后利用静态网页生成器将 markdown 文档转化为 html 文档。如此即可部署到静态网页托管的服务。

将 markdown 转化 html，我推荐使用 Hugo。

{{< image "2023-12/2023-12-06-create-your-own-free-blog-06.png" >}}

什么是 Hugo？Hugo 是 Go 编写的静态网页生成器，阅读它的 [官方文档](https://gohugo.io/documentation/)。

Hugo 利用的是 markdown 语法编写的文章渲染成 HTML 网页，它除了提供了大量的模版样式，还支持语法函数，实现站点的定制化。

Hugo 对于 blog 个人站点类的需求是特别适合，快速上手，简单易用。

### GitHub Page

有了网页内容，下面就是要找可提供免费部署的服务了。我推荐使用 GitHub Page。

什么是 GitHub Page？

GitHub Page，是GitHub 提供的免费云服务，它是以代码仓库形式托管 HTML 静态页面的服务。

GitHub Page 的优势在于，第一点是免费，无需购买机器。再者，使用云服务，能减少日常维护的时间，省心省力，还有一点，无需担心扩容问题。

## 其他组件

一个 blog，即使是个人博客，也不是只要展示内容即可的，其他如评论、图床、统计等也是必不可少的。好在，这些组件同样也皆有使用免费的方案。

### 评论 utterances

评论功能可使用 utterances 实现，一个轻量级的评论组件。它是基于 GitHub issue 实现，数据全部存储于 GitHub 中。

{{< image "2023-12/2023-12-06-create-your-own-free-blog-05.png" >}}

评论区的效果如下所示：

{{< image "2023-12/2023-12-06-create-your-own-free-blog-01.png" >}}

和其他一些组件相比，这也是一个纯净版的评论组件，干净整洁无广告。它的 GitHub 仓库地址：[utterances](https://github.com/utterance/utterances)

### 图床 jsdelivr CDN

对于博客而言，图片是必不可少的一部分，没有图片的博客，固然会少了一些趣味。我的方案中，图床是通过 jsdelivr CDN 加速访问 GitHub 中存储的图片。

[jsdelivr](https://github.com/jsdelivr/jsdelivr) 是一款用于开源项目的免费 CDN，它支持加速入 npm、ESM、GitHub、WordPress。

{{< image "2023-12/2023-12-06-create-your-own-free-blog-04.png" >}}

它的使用方式也非常简单。

一个例子，假设以 GitHub 仓库 poloxue/images 下 main 分支的名为 tag-zsh.png 的图片为例。如想通过 jsdelivr CDN 访问，只要以如下的形式访问即可。

```
https://cdn.jsdelivr.net/gh/poloxue/images/@main/tag-zsh.png
```

还是非常简单的。

本质上这使用的还是 GitHub 作为存储方案，这是势必要将 GitHub 的羊毛耗光啊。

### 统计能力 - goatcounter

博客统计分析能力，我选择的是 goatcounter 这个免费的项目。

[goatcounter](https://github.com/arp242/goatcounter) 是一个开源项目，它提供了两种方式，分别是使用它的托管服务或使用它的源码自建。

但它对个人用户，特别是开发人员还是非常易于使用。而且，它托管服务是免费的。

如下是它的界面效果：

{{< image "2023-12/2023-12-06-create-your-own-free-blog-02.png" >}}

相对于 Google Analytic，goatcounter 提供的能力没有那么强大，除了基本的访问数量统计，其他一些基本能力，如设备、语言、浏览器、操作系统的分析也都是支持的。

如下所示：
{{< image "2023-12/2023-12-06-create-your-own-free-blog-03.png" >}}

## 最后

本文介绍了一套搭建免费博客的可用组件方案，不是介绍详细的安装教程。希望在 2023 年，给于有兴趣搭建自己博客的朋友一点帮助吧。


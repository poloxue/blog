---
title: ""从头构建 Go Web 框架（一）：介绍
date: 2021-10-23:33:14+08:00
draft: true
---

本文写于 2014 年，对于 golang 十几年的发展历程而言，这已经是古董级别的一篇文章了，但通用的思想概念依然没有变。

我希望通过这个系列文章，让大家都现有的 Go Web 框架能有更深的认识。

访问：[原文地址](https://www.nicolasmerouze.com/build-web-framework-golang)，译文如下：

[Martini](https://github.com/go-martin/martini) 发布之后，迅速成为了最受大家欢迎的 Go 语言 Web 框架，且现在依旧是如此。但必须指出的是，它不符合常规习惯，非常慢，概念也有不足。它教了我们一堆错误的做法。但因为它上手容易，许多开发人员仍在使用。

似乎是存在即合理！

我决定写一系列文章，基于现有库从头编写组件来构建自己的 Web 框架。在开始前，我搜罗和阅读了市面上绝大部分关于如何编写 Go Web 应用的资料。我希望，通过系列文章能教授 Go Web 开发人员一些最佳实践，同时能提醒老 Go 开发人员什么才是 Web 开发的最佳实践。

注：每个项目都是具有独特性的，最佳实践并非适用于所有场景。

# 概要

本系列不仅能让你了解 Go 中 Web 开发的最佳实践，还会让你了解其他常见问题的解决方案及如何正确运用。

- [第 1 部分：简介](https://www.nicolasmerouze.com/build-web-framework-golang)
- [第 2 部分：Go 中间件：最佳实践和示例](https://www.nicolasmerouze.com/middlewares-golang-best-practices-examples)
- [第 3 部分：中间件数据共享](https://www.nicolasmerouze.com/share-values-between-middlewares-context-golang)
- [第 4 部分：第三方路由](https://www.nicolasmerouze.com/guide-routers-golang)
- [第 5 部分：使用 MongoDB 实现 JSON-API](https://www.nicolasmerouze.com/how-to-render-json-api-golang-mongodb)
- [附加福利：上传文件到 s3](https://www.nicolasmerouze.com/file-upload-web-service-golang-s3-aws)

## 框架该具备什么能力

市面主要有两种类型的 Web 框架。

一种是内置所有功能，类似 Rails 的框架，能帮你快速构建与引导项目。Go 中类似框架有：Beego 和 Revel。

另一种，类似于 Sinatra 的框架，提供路由和一些内置功能，但不会提供 ORM 等功能。 多数 Go 框架都采用了这种风格，如 Martini、Goji、gocraft/web 等。

## 框架与库

最近，发生了一场争论，主题是，我们究竟是使用框架还是使用库呢？

首先，我不反对框架，但 Go 有很棒的小包，能帮助我们非常容易地制作出自己的框架。特别是长期项目，自己动手是更明智的选择。如果你想学习围棋，最好了解一切是如何运作的。

争论详情查看以下文章：

[The Case for Go Web Frameworks](https://richardeng.medium.com/the-case-for-go-web-frameworks-a791fcd79d47)
[Go, frameworks, and Ludditry](https://dave.cheney.net/2014/10/26/go-frameworks-and-ludditry)
[Frameworks: Necessary for large-scale software](https://www.evanjones.ca/frameworks-necessary-for-large-scale-software.html)
[Framework vs Library](https://stephensearles.com/framework-vs-library/)

# 框架功能

我们将专注于后一种类型，因为涵盖 Beego 或 Revel 中的每个功能将意味着针对该主题编写一整本书！

框架主要包含 3 个部分，分别是：

- 一是 router，负责接收 request 并路由到指定 handler；
- 二是 middleware，handler 前后新增的复用性代码；
- 三是 handler 处理程序，负责处理请求并写入响应。

常见的中间件 Middleware：

- error/panic
- logging
- security
- sessions
- cookies
- body parsing

# Go Web 框架现状

现在，许多 Go Web 框架的问题在于，它们的组件仅适用于自身，是不可替换的。

假设现在有这样一些小包，只有路由器，或只有中间件系统，或者只有中间件，且它们彼此兼容，我们就可以将它们组合使用。 但现实是，这类项目数量极少，除了 Gorilla 之外，基本没什么知名度。当然，即使是 Gorilla，在 Github 上的追随者与其他大多数其他框架，依然少很多。

Go 的 Middleware 系统并没有统一的约定惯例。与 Ruby (Rack)、Node.js (Connect.js)、Clojure (Ring) 和其他语言不同。 每个框架，如 Negroni、Goji、Gin、gocraft/web 等，都在重新定义自己的中间件工作方式，重用开源中间件变得不可能。

什么才是新开发人员的最佳实践？

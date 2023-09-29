---
title: "从头构建 Go Web 框架（一）：介绍"
date: "2021-10-23T20:33:14+08:00"
comment: true
draft: false
tags: ["Golang", "Web"]
---

> 本系列文章写于 2014 年，相较于 golang 极短的发展历程，这已经是古董级别的一篇文章了，但 web 框架思想概念依然有效。系统通过这个系列文章，能让大家都现有 Go Web 框架有更深的认识。

本文是 "构建属于自己的 Web 框架" 系列文章中的第二篇，将介绍中间件的最佳实践。

- [第 1 部分：简介](https://www.poloxue.com/posts/2021-10-23-build-your-own-webframework-in-golang)，[Build Your Own Web Framework In Go](https://www.nicolasmerouze.com/build-web-framework-golang)
- [第 2 部分：Go 中间件：最佳实践和示例](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2)，[Part 2: Middlewares in Go: Best practices and examples](https://nicolasmerouze.notion.site/Part-2-Middlewares-in-Go-Best-practices-and-examples-32f41ae0e21b435c86cf9dd38bf0ff65)
- [第 3 部分：中间件数据共享]()，[Part 3: Share Values Between Middlewares](https://www.nicolasmerouze.com/share-values-between-middlewares-context-golang)
- [第 4 部分：第三方路由]()，[Part 4: Guide to 3rd Party Routers in Golang](https://nicolasmerouze.notion.site/Part-4-Guide-to-3rd-Party-Routers-in-Go-8ddcca5c360b4539a601ae383c9d7e5d)
- [第 5 部分：使用 MongoDB 实现 JSON-API]()，[How to implement JSON-API standard in MongoDB and Go](https://nicolasmerouze.notion.site/Part-5-How-to-implement-JSON-API-standard-in-MongoDB-and-Go-a3daeead140846e4a6ae1e3c01b47f52)
- [附加福利：上传文件到 s3]()，[Bonus: File Upload REST API with Go and Amazon S3](https://nicolasmerouze.notion.site/Bonus-File-Upload-REST-API-with-Go-and-Amazon-S3-1130fbacad7442c5b0a7df9320d792b4)

[Martini](https://github.com/go-martin/martini) 发布之后，迅速成为了最受大家欢迎的 Go 语言 Web 框架，且现在依旧是如此。但必须指出的是，它不符合常规习惯，非常慢，概念也有不足。它教了我们一堆错误的做法。但因为它上手容易，许多开发人员仍在使用。

似乎是存在即合理！

故而，我决定写一系列文章，基于现有库从头编写组件来构建自己的 Web 框架。在开始前，我搜罗和阅读了市面上绝大部分关于如何编写 Go Web 应用的资料。我希望，通过系列文章能教授 Go Web 开发人员一些最佳实践，同时能提醒老 Go 开发人员什么才是 Web 开发的最佳实践。

# 概要

本系列不仅能让你了解 Go 中 Web 开发的最佳实践，还会让你了解其他常见问题的解决方案，以及如何正确运用其他框架。

## 框架该具备什么能力

市面主要有两种类型的 Web 框架。

一种是内置所有功能，类似 Rails 的框架，能帮你快速构建与引导项目。Go 中类似框架有：Beego 和 Revel。

另一种，类似于 Sinatra 的框架，提供路由和一些内置功能，但不会提供 ORM 等功能。 多数 Go 框架都采用了这种风格，如 Martini、Goji、gocraft/web 等。

## 框架与库

最近，发生了一场争论，主题是，究竟是该使用框架还是使用库呢？

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

什么才是开发人员的最佳实践？请阅读下一篇，[从头构建 Go Web 框架](https://www.poloxue.com/posts/2021-10-28-build-your-own-webframework-in-golang-part-2)。

博文地址：[从头构建 Go Web 框架](https://www.poloxue.com/posts/2021-10-23-build-your-own-webframework-in-golang/)，译：[Build Your Own Framework in Golang](https://www.nicolasmerouze.com/build-web-framework-golang)。


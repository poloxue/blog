---
title: "从头构建 Go Web 框架（二）：中间件"
date: 2021-10-28T13:35:05+08:00
comment: true
draft: true
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




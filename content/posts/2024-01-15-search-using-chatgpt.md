---
title: "利用 ChatGPT 高效搜索：举一反三的思考方式，高效查找解决方案"
date: 2024-01-15T19:07:52+08:00
draft: false
comment: true
description: "本文只是我的一些尝试，基于 ChatGPT 实现系统化快速搜索某编程语言的特定领域相关包或者基于其他语言类推荐落地方案的尝试。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-15-search-using-chatgpt-01.png)


本文只是我的一些尝试，基于 ChatGPT 实现系统化快速搜索某编程语言的特定领域相关包或者基于其他语言类推荐落地方案的尝试。

这篇文章中描述的方式不一定是好方式，但应该会有一定的启示作用吧。让 ChatGPT 为我们的开发效率添砖加瓦。

## 基础思路

在学习和使用一门新的编程语言的过程中，找到合适的包对于解决特定问题至关重要。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-15-search-using-chatgpt-02.png)

传统上，还是主要依赖搜索引擎和社区资源来寻找这些包，但这个过程往往既耗时又充满挑战。现在，有了 ChatGPT，这一切都有了新的解决方案。

传统的搜索方法通常基于关键词搜索，但这种方法往往返回只是简单列举，我们要从大量不相关的结果找出我们想要的内容，需要花费大量时间去筛选。

ChatGPT 提供了一种更高效的方式：通过与 AI 对话来精确描述我们的需求，从而获得更加精准回答。不过我想说，不要像利用传统的搜索引擎一样利用 ChatGPT，ChatGPT 会思考。但毫无疑问，再聪明的好苗子，也是需要引导的。

## 举一反三全面了解 GO 的 Web 框架

为了最大化 ChatGPT 的效用，我采用固定句式来描述我的需求，这么做的目标主要是为了引导它思考。

例如，当寻找 Golang 的 web 开发框架时，我不是简单地问 “给我推荐一些 web 开发框架”，而是通过类比的方式地询问，想全访问了解不同情况下使用什么 HTTP 框架。



我知道在使用 Golang 的数据库时，可以直接使用 sql/database + 驱动连接数据库，也可以使用 ORM 方案，如 sqlx 或 GORM。HTTP 一般也是如此，有底层核心层，上层的用户体验较好的框架。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-15-search-using-chatgpt-03.png)

我按这个思考简单组织了一些语言。如下所示：

> 按我这个句式说下关于 Go 的 xxx 相关内容:
> Go 语言中，处理数据库的包通常分为三个主要层次：底层实现、数据库驱动以及上层的 ORM（对象关系映射）框架。首先，底层核心包是 `sql/database`，它为数据库操作提供了基础的接口和功能。接着，针对不同数据库类型的驱动包各不相同，例如，用于 MySQL 的驱动包是 [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)。最后，在更高的层面上，有基于如 `sqlbuilder` 或 `activerecord` 模式的实现，Go 语言中的相应包包括 [ent](https://github.com/ent/ent) 和 [gorm](https://github.com/go-gorm/gorm)，这些包提供了更高级别的数据库操作和抽象。

这个写的稍微有点长，应该可以组织的更加精简一点。我主要想表达：

通过这种方式，引导 ChatGPT 去像人来一样地思考，举一反三，深入地理解需求。而不是像以前的 AI 一样，机械呆滞地返回固定问题的答案。

有些时候，语言直接表达的效果是没有这种类比效果好的。不仅仅这种方式适用于 ChatGPT，如果你和人类这样对话，也是更加容易让对方理解。

我就是通过这个方式，比较全方位了解 Go 语言不同场景下 Web 框架包的。如下是我具体的问题：

> 按我这个句式推荐下 Go 的 Web 框架。[句式如上，省略]*

GPT 的回答如下所示：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-15-search-using-chatgpt-04.png)

> 在 Go 语言中，HTTP 框架的组织也可以分为几个主要层次，类似于处理数据库的包的结构。首先，底层核心包是 `net/http`，提供了构建 HTTP 服务器和客户端的基本工具，如路由、请求处理和发送响应。接下来，针对不同需求的 HTTP 中间件和路由工具，例如 [Gorilla Mux](https://github.com/gorilla/mux) 提供了高级的路由功能，而 [Negroni](https://github.com/urfave/negroni) 提供了易用的中间件管理。在更高的层次上，存在着基于如 `mvc` 或 `restful` 模式的全功能 Web 框架，如 Go 语言中的 [Gin](https://github.com/gin-gonic/gin) 和 [Echo](https://github.com/labstack/echo)，它们提供了路由、中间件、模板渲染等更丰富的功能，以支持复杂的 Web 应用开发。此外，还有针对特定性能优化的 HTTP 工具，例如 [Fasthttp](https://github.com/valyala/fasthttp)，它专注于提供高性能的 HTTP 服务，适用于处理大量并发请求。这样的层次划分帮助开发者根据具体需求选择合适的 Go 语言 HTTP 框架和库。


## 系统方案建议 - 让 ChatGPT 推断

类似的，我这里再举个例子。针对同一个问题，可能实现思路是一样的，但实现的工具就具体情况而异。这时候，我们可以通过提供对这个问题的一种方案，让 ChatGPT 衍生，提供我们想要的另一个场景条件的实施方案。

一个例子，比如我们习惯于使用 GO 开发 web 项目，也了解如何实施。但是现在公司希望转入 Python 或者 rust，如何最快的找到对标的实现呢？

我们可以说：

> 在 Go Web 开发时，要用到的库有：Web 框架使用 gin、 orm 使用 gorm，用户权限认证使用 [golang-jwt/jwt](https://github.com/golang-jwt/jwt)，依次类推 python 开发 Web 项目的推荐库

ChatGPT 返回内容：

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-15-search-using-chatgpt-05.png)


> 对于Go Web开发，您提到的库选择是很不错的。以下是一些在Python Web项目中常用的库的推荐：
> 
> 1. Web框架：
>    - Flask：Flask是一个轻量级的Python Web框架，适用于小型到中型的项目。
>    - Django：Django是一个功能强大的Python Web框架，适用于大型项目和快速开发。
> 
> 2. ORM（对象关系映射）：
>    - SQLAlchemy：SQLAlchemy是一个流行的Python ORM库，它提供了多种数据库后端的支持，并具有强大的查询和模型定义功能。
> 
> 3. 用户权限认证：
>    - Flask-Login：用于用户会话管理和身份验证的Flask扩展。
>    - Django-Allauth：Django的全功能身份验证和账户管理解决方案。
> 
> 4. JWT（JSON Web Tokens）：
>    - PyJWT：PyJWT是Python中处理JWT的库，可以用于用户身份验证和授权。
> 
> 这些库在Python Web开发中广泛使用，可以根据项目的需求选择合适的库。希望这些推荐对您有所帮助。如果您需要更多信息或有其他问题，请随时提问。

它这里面还多了一部分关于权限认证，可能还是我的问题提的有点瑕疵，或者 chatgpt 自己的问题。反正是，多结果总觉少结果要好。

rust 是不熟悉的语言，我最近在尝试学习，同样的思路，还可以问 rust 的 web 开发套件。

chatGPT 回答如下：

> 当使用Rust进行Web项目开发时，以下是一些推荐的库和框架：
> 
> 1. Web框架：
>    - Actix-web：Actix-web是一个高性能的、基于异步的Web框架，适用于构建快速、可伸缩的Web应用程序。
>    - Rocket：Rocket是一个易于使用的Web框架，它提供了类型安全的路由和请求/响应处理。
> 
> 2. ORM（对象关系映射）：
>    - Diesel：Diesel是Rust的ORM库，它提供了类型安全的数据库查询和模型定义功能。
>    - sqlx：sqlx是一个异步SQL库，允许您使用Rust进行数据库操作。
> 
> 3. 用户权限认证：
>    - Keycloak：Keycloak是一个开源的身份和访问管理解决方案，可以用于用户身份验证和授权。
>    - Auth0：Auth0是一个身份验证和授权服务，可以轻松集成到Rust应用程序中。
> 
> 4. JWT（JSON Web Tokens）：
>    - jsonwebtoken：jsonwebtoken是Rust中处理JWT的库，可用于身份验证和令牌生成。
> 
> 这些库和框架可以帮助您构建高性能、安全的Rust Web应用程序。根据项目的需求和您的偏好，选择适合的库和框架进行开发。如果您需要更多信息或有其他问题，请随时提问。

不知道这个答案质量，你觉得如何呢？毫无疑问，这个答案，吊打搜索引擎。我这里只是简单举例，你还可以提供更复杂的需求，让它提供更多的推荐。

## 我的一些微末思考

通过上面的这些案例，我主要想说  ChatGPT 能像人类一样思考，但它的思考的模式是我们提供的。

这里面有一个非常简单的道理，如果你把 ChatGPT 想象是一个真正的人类，即使这个人类一个非常牛逼的大佬，但是如果他不是从教育你的角度出发，就不会说一些无关内容，一些的互动都是基于你的主动性，你的输入。如果你的问的简单，它必然是回答的简单。

同理，如果你想让 ChatGPT 思考，你需要告诉它如何思考。这就是你要提供的信息，这个信息最能体现你的个人能力了。ChatGPT 能思考，但是基于的内容是你的输入的扩展。
## 结论

我想说 ChatGPT 不仅是一个对话工具，更是一个强大的技术资源搜索助手，或者说，它可以成为你的朋友，导师，关键在于你是否了解如何与它交流，让它如何去做。

它改变了寻找技术解决方案的方式。我通过简单的对话，让它思考推导，而不是像以前的智障 AI 或搜索引擎一样针对问题固定返回某个回答。

这种结合 AI 技术的搜索方法不仅节省了我们的时间，还为我们提供了更加精确和深入的技术见解，而非是简单的列举。

博文地址：[利用 ChatGPT 高效搜索：举一反三的思考方式，高效查找解决方案](https://www.poloxue.com/posts/2024-01-15-search-using-chatgpt)

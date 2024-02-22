---
title: "一个基于差异同步数据库结构的工具 - Skeema"
date: 2024-02-22T17:29:22+08:00
draft: true
comment: true
description: "本文将继续介绍数据库 schema 数据库同步工具。今天，推荐是的一个基于差异方式实现数据库 schema 迁移的工具库 - skeema，同样也是基于 Go 实现。"
---

本文继续介绍数据库 schema 同步工具，我前面已经写了两篇这个主题的文章。

今天，推荐是的一个基于差异实现数据库 schema 迁移的工具库 - skeema，同样由 Go 实现。

## 背景

我的上家公司时，有名架构师开发了一个类似于 skeema 的工具，也是基于差异同步表结构的，区别在于那个工具使用的 yaml 声明表结构定义的。我当时就想在市面上找一个类似的工具，但没找到。好在通过 ChatGPT，搜索效率提升很多，不过也是一步步的引导才让我找到这个工具。

## 概述

Skeema 基于差异同步数据库 schema，我们关心表结构的终态即可。用 skeema 可使不同环境间同步数据库 schema 变得很简单。

Skeema 支持 linter 识别 SQL 语句，方便我们将其集成到 CICD 中提升 Schema 质量。还有，它默认是禁用了一些不安全的数据库操作，如删表删字段等信息。

如果要说缺点，它现在只支持 MySQL 和 MariaDB。

## 安装

Skeema 的安装过程直接了当。如果你使用的是 MacOS，可以通过Homebrew来安装：

```bash
brew install skeema/tap/skeema
```

或是也可通过 `go get` 安装，但 Go 的版本要求在 v1.21+。

```bash
$ go install github.com/skeema/skeema@v1.11.1
```

对于其他平台，可以从Skeema的GitHub页面下载二进制文件，并按照文档指引进行安装。

## 使用

Skeema 的使用可概括为 5 个核心步骤：

- 通过 skeema init 下载初始建表 SQL；
- 基于初始 SQL 按需求修改文件；
- 使用 skeema diff 在提交前确认差异；
- 使用 skeema lint 检查 SQL 是否满足 linter 规则；
- 使用 skeema push 推送 SQL，变更数据库 schema。

我将按这个步骤展开介绍。

### 初始化

首先，通过 skeema init 实现基于现有的数据库结构初始我们的目录，生成初始建表语句。

示例如下：

```bash
$ skeema init -h 127.0.0.1 -uroot -ppassword --schema blog -d .
```

`--schema` 用于指定目标库名称，如果没有指定，会生成实例下所有库的建表 SQL。而 `-d/--dir` 用于指定目标目录。

现在，回车确认和输入密码执行命令。

由于我这个 blog 数据库没有任何内容，只会在当前目录下生成一个 `.skeema` 文件，它也就是 `skeema` 的配置文件。

打开它，查看内容如下：

```toml
default-character-set=utf8mb4
default-collation=utf8mb4_0900_ai_ci
generator=skeema:1.11.1-community
schema=blog

[production]
flavor=mysql:8.3
host=127.0.0.1
port=3306
user=root
```

如果想在接下来执行其他命令时，省略掉 `-p` 输入密码的步骤，可加上密码配置。

```toml
...
port=3306
user=root
password=password
```

这一步会在指定目录下创建数据库架构的本地表示。

而如果我没有指定 `--schema` 选项，将会把数据库中所有库表初始化到我的目录下。

```bash
$ ls -a
.skeema  article  blog  core
```

### 生成差异

现在尝试做一些变更，article 库下的有一个名为 comments 的表，它的建表语句如下所示：

```sql
CREATE TABLE `comments` (
  `id` int NOT NULL,
  `post_id` int NOT NULL,
  `author_name` int NOT NULL,
  `comment` text NOT NULL,
  `update_at` timestamp NULL DEFAULT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

我将在其中添加一个 `author_id` 字段，修改的建表语句如下所示。

```sql
CREATE TABLE `comments` (
   ...
  `author_id` varchar(255) NOT NULL,
   ...
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```

现在，让我们使用 `skeema diff` 查看差异。

```bash
$ skeema diff
2024-02-22 18:53:09 [INFO]  Generating diff of 127.0.0.1:3306 article vs
                            /Users/jian.xue/demo/databasemigration/skeema2-demo/article/*.sql
-- instance: 127.0.0.1:3306
USE `article`;
ALTER TABLE `comments` ADD COLUMN `author_id` int NOT NULL AFTER `post_id`;
2024-02-22 18:53:09 [INFO]  127.0.0.1:3306 article: diff complete
...
```

从上得到在 comments 新增 author_id，要执行的 SQL 是：

```sql
ALTER TABLE `comments` ADD COLUMN `author_id` int NOT NULL AFTER `post_id`;
```

这个命令成功输出了当前目录下建表语句的结构与数据库中表结构的差异。通过查看这个差异，即可确认是否是我们预期的 SQL。

但问题是，和代码审阅一样，如果所有问题都是依靠人来检查，则很难最大限度降低问题发生的可能性。在代码审阅的流程中，通过会加入 linter 工具，SQL 同样也可以有这流程。好在，skeema 也提供了类似这样 lint 能力。

### Linter

接下来，介绍如何基于 skeema 实现 linter 能力。

Skeema 还内置了一个linter 工具，帮助确保表结构定义遵循最佳实践和常见规范。

运行以下命令即可：

```bash
skeema lint
```

这将检查架构文件中的潜在问题，并给出建议。

### 应用变更

确定要应用这些变更后，可以使用`push`命令来更新数据库架构：

```bash
skeema push
```

这会将本地架构变更应用到数据库中，同步所有差异。

## 自动化

如果公司提倡自动化，CICD 流程等，审核流程较为轻量，是否可将其直接集成到流水线中，基于生成的 SQL 自动化推送到生产环境。

我之前看到过一个实际的生产事故案例。

项目早期的数据量不大，提倡效率，数据库的变更就是采用直接发布 CICD 流程变更的，CICD 的控制权也都在开发手里。后来，随着表中数据的量级变大，某一次自动更新中包含新增表字段时，导致了长时间的锁表，用法访问页面出现大量错误。

这件事后，公司的 DBA 团队就直接把这个工具弃用了，每次都是开发提交类似 `CREATE`、`ALTER` SQL 给 DBA 审核执行。一旦发布，开发、测试、运维、DBA 全员到齐，少一个人都没有办法发布。

我不想讨论是这流程形成的原因，它不仅仅是技术问题，还有公司文化，甚至是政治问题，这是小兵力所不及的。

## 总结

Skeema 作为一个基于差异同步数据库 schema 的工具，简单强大。它简化了数据库 schema 的管理。使得我们无论是开发新功能时管理架构变更，还是在多环境中同步数据库架构，Skeema都能提供有效的支持。

如果你和我曾经一样，为不同环境间的数据库表结构管理同步感到头痛，试试 Skeema，或许你会喜欢这种方式。


---
title: "一个基于差异同步数据库结构的工具 - Skeema"
date: 2024-02-22T17:29:22+08:00
draft: true
comment: true
description: "本文将继续介绍数据库 schema 数据库同步工具。今天，推荐是的一个基于差异方式实现数据库 schema 迁移的工具库 - skeema，同样也是基于 Go 实现。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-28-database-migration-tools-skeema-in-golang-01.png)

本文是 GO 三方库推荐的第 5 篇，继续介绍数据库 schema 同步工具，我前面已经写了两篇这个主题的文章。系列查看：[Golang 三方库](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0MzE2NTY2MA==&action=getalbum&album_id=3302384940181110785#wechat_redirect)。

今天，推荐是的一个基于差异实现数据库 schema 迁移的工具库 - skeema，同样由 Go 实现。

## 背景

我的上家公司时，有名架构师开发了一个类似于 skeema 的工具，也是基于差异同步表结构的，区别在于那个工具使用的 yaml 声明表结构定义的。我当时就想在市面上找一个类似的工具，但没找到。好在通过 ChatGPT，搜索效率提升很多，不过也是一步步的引导才让我找到这个工具。

## 概述

Skeema 是基于差异同步数据库 schema，这让我们只要表结构的终态就行。 还有，skeema 支持在不同环境间同步数据库 schema。

Skeema 支持 linter 识别 SQL 语句，方便我们将其集成到 CICD 中提升 Schema 质量。还有，它默认是禁用了一些不安全的数据库操作，如删表删字段等操作。

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
  PRIMARY KEY (`id`),
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

Skeema 内置了一个linter 工具，检查表结构定义语句，防止一些常规的问题。

运行命令：

```bash
skeema lint
```

Skeema 提供有大量规则选项，有兴趣去看下它的官方文档：[Option Reference](https://www.skeema.io/docs/options/)。

如下是我列举一些常见的大家感兴趣的规则：

```toml
# 设置默认的字符集，推荐使用 utf8mb4 以支持全字符集
default-character-set=utf8mb4

# 确保所有表和列使用推荐的字符集
lint-charset=error
allow-charset=utf8mb4

# 确保所有表使用推荐的存储引擎，如 InnoDB
lint-engine=error
allow-engine=innodb

# 推荐使用合适的数据类型作为主键，通常是整数类型
lint-pk-type=error
allow-pk-type=int,bigint

# 根据团队策略对外键使用进行限制，如果避免使用外键以提升性能
lint-has-fk=warning

# 确保自增列使用合适的数据类型，避免未来可能的整数溢出
lint-auto-inc=error

# 避免重复或冗余的索引
lint-dupe-index=error

# 如果应用策略限制了存储过程和函数的使用
lint-has-routine=warning

# 确保表使用一致的命名规则，如全部小写
lint-name-case=error
```

现在尝试对 `comments` 做一些变更，测试能检查出主键类型错误和索引问题。

```sql
CREATE TABLE `comments` (
  `id` CHAR(32) NOT NULL,
  `post_id` int NOT NULL,
  `author_name` int NOT NULL,
  `comment` text NOT NULL,
  `update_at` timestamp NULL DEFAULT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_post_id` (`post_id`),
  KEY `idx_post_id_duplicate` (`post_id`), -- 重复的索引，与 idx_post_id 完全相同
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

执行如下命令：

```bash
$ skeema lint
2024-02-23 15:55:55 [INFO]  Linting /xxxx/skeema-demo
2024-02-23 15:55:55 [INFO]  Linting /xxxx/skeema-demo/article
2024-02-23 15:55:55 [INFO]  Wrote
                            /xxxx/skeema-demo/article/comments.sql
                            (500 bytes)
2024-02-23 15:55:55 [ERROR] /xxxx/skeema-demo/article/comments.sql:2:
                            Column id of table `comments` is using data type char(32), which
                            is not configured to be permitted in a primary key. The following
                            data types are listed in option allow-pk-type: int.
2024-02-23 15:55:55 [ERROR] /xxxx/skeema-demo/article/comments.sql:10:
                            Indexes idx_post_id_duplicate and idx_post_id of table `comments`
                            are functionally identical.
                            One of them should be dropped. Redundant indexes waste disk space,
                            and harm write performance.
2024-02-23 15:55:55 [ERROR] Found 2 errors
```

提示索引类型错误：

```bash
Column id of table `comments` is using data type char(32), which
is not configured to be permitted in a primary key. The following
data types are listed in option allow-pk-type: int.
```

提示重复索引错误：

```bash
Indexes idx_post_id_duplicate and idx_post_id of table `comments`
are functionally identical.
One of them should be dropped. Redundant indexes waste disk space,
and harm write performance.
```

skeema 有社区版和商业版，有一些能力检查规则要商业版支持。

### 应用变更

skeema 使用的最后一步是应用这些变更，使用 `skeema push` 命令即可将更新应用到数据库。这个没啥可介绍的了。

## 多环境配置

skeema 支持多环境的不同配置，上面的配置实例中其实已经能看出了。其中有一个 production 段。

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

如我增加一个 dev 环境，命令如下：

```bash
$ skeema add-environment dev --host 127.0.0.1 -uroot -ppassword
```

配置如下：

```bash
[dev]
flavor=mysql:8.3
host=127.0.0.1
port=3306
user=root
password=password
```

使用时，只要在命令后加上环境名称即可，如：

```bash
$ skeema push dev
```

还有，在不同环境下使用不同 lint 规则，这也是可以的。

## 安全操作

还有一定要说明，记住 skeema 默认是不允许一些非安全的操作，如删表、列其它如修改导致数据截断啥的等等操作。这也是很符合实际场景的。不然，因为某操作，导致删掉一些数据就完犊子了。

如果有非安全操作的需求，如在开发环境，不喜欢保留一些开发期间临时创建的表，配置 `allow-unsafe=true` 即可。

```toml
[dev]
...
allow-unsafe=true
```

## 总结

Skeema 作为一个基于差异同步数据库 schema 的工具，简单强大。它简化了数据库 schema 的管理。使得我们无论是开发新功能时管理架构变更，还是在多环境中同步数据库架构，Skeema都能提供有效的支持，都不再复杂。

最后，如果你和我曾经一样，为不同环境间的数据库表结构管理同步感到头痛，试试 Skeema，或许你会喜欢这种方式。


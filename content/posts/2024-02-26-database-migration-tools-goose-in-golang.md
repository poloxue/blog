---
title: "Goose: Go 实现的增量迁移数据库结构的工具"
date: 2024-02-20T16:15:01+08:00
draft: true
comment: true
description: "我将以这个数据库结构迁移为基础，推荐两个 Go 实现的数据结构同步工具，它们基于的是两种完全不同的实现方式：增量和差异。"
---

日常的项目开发中，我们基本每天都要与数据库打交道。而与数据库相关的任务中，有一件繁琐但不得不做的任务：在不同环境间同步数据表结构？

我将以这个为主题为基础，推荐两个 Go 实现的数据结构同步工具，它们基于的是两种完全不同的实现方式：增量和差异。

今天，介绍第一个工具 — Goose，它采用的是增量方式管理数据库结构。

我不知道其他人的情况，我工作的很长一段时间内，接触到的这类工具都是采用增量方式实现的。文末，我会说明增量迁移的一些缺点和可能的解决方案。

## 为什么选择 Goose？

首先，市场上已经存在了那么多数据库迁移工具，如 Flyway、Liquibase 和 Alembic。它们一般也都是采用增量方式管理数据库迁移。

在这众多选项中，为什么要用 Goose 呢？我觉得最主要原因就是，它简单易上手，我们能迅速掌握并开始迁移任务。

## 快速一览 Goose 功能特性

**多数据库支持**：Goose 可与多种数据库系统一起使用，包括但不限于 MySQL, PostgreSQL, SQLite, 和SQL Server。

**向前和向后迁移**：支持向前（up）和向后（down）迁移，不仅可以通过迁移来更新数据库结构，还可以回滚到之前的状态。

**编程语言灵活性**：Goose 最初是用 Go 编写的，但它支持在迁移文件中使用纯 SQL，这就拓宽了它的目标用户群体。

**命令行接口**：Goose 提供了一个简单的命令行接口（CLI），使执行迁移操作更简单直观。这篇文章中的演示都是基于命令接口。 

**版本控制**：Goose 的每次迁移都会记录版本号，使数据库的版本控制变得清晰。版本号是基于时间生成的。

**环境配置**：我简单改造了 goose 让它支持根据不同的环境（如开发、测试、生产）配置和应用不同的迁移策略。

## 安装配置

安装 Goose 相当简单。首先，确保你已经安装了 Go。然后，通过下面命令安装 Goose：

```sh
go install github.com/pressly/goose/v3/cmd/goose@latest
```

这条命令会将 goose 命令安装到 GOBIN  目录下。到此，就顺利安装成功了。

## 演示案例

接下来，我将通过案例逐步演绎 Goose 的使用，基于 Goose cli 命令管理这些 SQL 脚本。

### 创建迁移

假设，现在要为一个项目添加一个用户表。第一步就是创建一个 SQL 迁移脚本。

迁移脚本创建，命令如下：

```bash
$ goose create create_users_table sql
2024/02/20 17:52:58 Created new file: 20240220095258_create_users_table.sql
```

命令的最后一个参数 sql 表示创建的 SQL 脚本，如果没有这个参数，默认创建的是 Go 脚本。

默认脚本内容：

```sql
-- +goose Up
-- +goose StatementBegin
SELECT 'up SQL query';
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
SELECT 'down SQL query';
-- +goose StatementEnd
```

这个示例脚本包含了两部分：同步（Up）和回滚（Down）。其中的 `- +goose Up` 标记了接下来的 SQL 语句是用于同步迁移的。而 `-- +goose Down` 标记了接下来的 SQL 语句是用于回滚的。

我们用创建 `users` 表的 SQL 替换其中的 `SELECT 'up SQL query`，用删除 users 表的 SQL 替换 `SELECT 'down SQL query`：

```sql
-- +goose Up
-- +goose StatementBegin
CREATE TABLE users (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(64) NOT NULL,
    password CHAR(32) NOT NULL,
    email VARCHAR(255) UNIQUE  NOT NULL
);
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE if EXISTS users;
-- +goose StatementEnd
```

我们使用 Goose 的 up 执行同步迁移操作。

```sh
$ goose mysql "root:password@/core?parseTime=true" up
2024/02/20 18:07:57 OK   20240220095258_create_users_table.sql (21.8ms)
2024/02/20 18:07:57 goose: successfully migrated database to version: 20240220095258
```

这条命令会应用所有未执行的迁移脚本，创建数据库 users 表结构。Goose 会跟踪每次迁移的状态，确保每个迁移只被执行一次。

如果打开数据库，查看表的创建结果，你会发现，数据库中多了一个非我们预期内的表 - goose_db_version。这个表就是 goose 用来追踪每次迁移的状态信息。

```mysql
mysql> show tables;
+------------------+
| Tables_in_core   |
+------------------+
| goose_db_version |
| users            |
+------------------+
2 rows in set (0.01 sec)
mysql> select * from goose_db_version;
+----+----------------+------------+---------------------+
| id | version_id     | is_applied | tstamp              |
+----+----------------+------------+---------------------+
|  1 |              0 |          1 | 2024-02-20 10:07:57 |
|  2 | 20240220095258 |          1 | 2024-02-20 10:07:57 |
+----+----------------+------------+---------------------+
```


如果想回滚之前创建的表，使用命令 `down` 即可回滚。

```bash
$goose mysql "root:password@/core?parseTime=true" down
2024/02/20 18:21:38 OK   20240220095258_create_users_table.sql (16.7ms)
```

Goose 其他的一些命令如下所示。当使用 Goose 进行数据库迁移时，你可以使用它们。

```
Commands:
 up-by-one       Migrate the DB up by 1
 up-to VERSION   Migrate the DB to a specific VERSION
 down-to VERSION Roll back to a specific VERSION
 redo            Re-run the latest migration
 reset           Roll back all migrations
 status          Dump the migration status for the current DB
 version         Print the current version of the database
 fix             Apply sequential ordering to migrations
 validate        Check migration files without running them
```

快速一览：

1. **up-by-one：** 逐步迁移数据库，只迁移一个版本。
2. **up-to VERSION：** 迁移数据库到指定的版本。
3. **down-to VERSION：** 回滚数据库到指定的版本。
4. **redo：** 重新运行最新的迁移。
5. **reset：** 回滚所有的迁移，删除所有已应用的迁移。
6. **status：** 输出当前数据库迁移的状态。它会显示已生效版本和未生效版本。
7. **version：** 打印当前数据库的版本号。
8. **fix：** 将迁移文件转换为顺序号顺序。确保按照顺序号的执行。
9. **validate：** 检查迁移文件的有效性，但不运行它们。

这些命令增加了增量方式类的迁移工具的灵活性和可控性，我们可按需执行特定的操作，灵活管理版本和迁移历史。

## 管理多环境

前面的演示，总是要在命令行上指定数据库类型和连接参数才能连接真实环境数据库，即麻烦又容易出错。

```bash
$ goose mysql "root:password@/core?parseTime=true" up
```

好在，Goose 提供了通过环境变量配置数据库的连接信息。

```bash
# 数据库类型，如 MySQL、PostgreSQL 等
GOOSE_DRIVER=DRIVER 
# 连接信息，如 MySQL root:password@/core?parseTime=true
GOOSE_DBSTRING=DBSTRING 
```

现在就可以直接执行如 `goose up`、`goose down` 等命令。

另外，我为了让 Goose 支持多环境配置，开发了一个简单的 shell 脚本，从配置中获取环境信息，设置到这两个环境变量上。

配置的形式如下：

```yaml
dev:
    driver: mysql
    open: user:password@/dbname?parseTime=true
test:
    driver: mysql
    open: user:password@/test_dbname?parseTime=true
prod:
    driver: mysql
    open: user:password@/prod_dbname?parseTime=true
```

在上面的例子中，我们定义了三个环境：`dev`、`test` 和 `prod`。每个环境都有自己的数据库连接字符串。

支持环境管理的命令脚本源码如下所示，我将其命名为 `goosem`。

```bash
#!/bin/bash

if ! command -v yq &> /dev/null
then
    echo "yq could not be found. Please install yq:\n\t`brew install yq`."
    exit 1
fi

if ! command -v goose &> /dev/null
then
    echo "goose could not be found. Please install goose:\n\t`brew install goose`."
    exit 1
fi

if [ "$#" -lt 1 ]; then
    echo "Usage: $0 <environment> [goose commands]"
    exit 1
fi

ENV=$1
CONFIG_FILE="dbconf.yml"

DRIVER=$(yq e ".$ENV.driver" $CONFIG_FILE)
DBSTRING=$(yq e ".$ENV.open" $CONFIG_FILE)

if [ "$DRIVER" == "null" ] || [ "$DBSTRING" == "null" ]; then
    echo "Configuration for environment '$ENV' not found."
    exit 1
fi

export GOOSE_ENV=$ENV
export GOOSE_DRIVER=$DRIVER
export GOOSE_DBSTRING=$DBSTRING

shift

goose "$@"
```

现在，我们可以通过如下命令使用 Goose 了。

```sh
goosem dev up
```

这个脚本里面依赖了一个 yaml 解析工具，名为 yq，要提前安装下：

```
$ brew install yq
```

现在，Goose 就有了支持多环境的能力了。非常简单。

如果你有按环境执行不同的迁移脚本的需求，可通过使用 Go 迁移脚本，在其中添加环境检查的逻辑来实现这一点。

举例来说，你可以在 Go 迁移脚本中检查是否是 prod 环境决定是否执行某部分迁移逻辑。

```go
if os.Getenv("GOOSE_ENV") == "prod" {
  // ...
}
```

这有个前提，你要熟悉 GO 这门编程语言。

## 增量模式的缺点

Goose 确实提供了管理数据库结构的简便方法，但跟着它走下来，我发现了几个挺头疼的地方。

首先，每次改动都得写一个新的“迁移文件”。岁项目发展，这些文件的数量会异常大，管理和使用它们就变成了一项挑战。而且，如果想要搭建一个新环境，得从头到尾跑一遍所有的迁移。我亲身经历过，中途某个环节出了问题，结果是得一个个文件修，修来修去，改了一堆文件。我快崩溃了。

在处理多分支时，数据库迁移可能引发合并冲突，搞得人头大。这顺序依赖的问题让事情更加复杂。

如果是多环境，我经常碰到另一个问题：开发过程中频繁地修改字段。这时，要么我直接手动改SQL，要么还是依赖这类工具来改。如果手动改，SQL 变更可能就没记录下来，导致环境间不一致，或者某些修改直接就丢了。

要环境间保持一致，就得不断地更新版本来回滚之前的修改。如果生产环境也跑这套脚本，就意味着同样的操作要重复执行多次，迁移文件也会越来越多，管理起来就更头疼了。

Goose 虽然能让我们灵活执行任意版本的迁移，但这操作太繁琐了，而且跟 CI/CD 工具集成起来也麻烦，毕竟太依赖手动操作。

对这个问题，我首先想的是，有没有什么工具提供类似 git squash 的能力，把一段时间的 SQL 改动合并成一个文件呢？这样至少文件少点，依赖问题也能减轻些。不过，貌似大多数增量迁移工具都不支持这个。当然，即使支持也支持缓解了问题，并没有解决问题。

如果走手动合并的路线，那版本控制和环境隔离就成了大问题，需要精心规划，确保通过多人审查来控制发布。

好吧，对于数据库结构，我关心只是最终的结构，如果能基于结构差异，生成差异 SQL，采用打补丁的方式迁移不是更好的方法吗？

## 最后

本文介绍了一个 GO 开发的基于增量实现数据结构迁移的工具 goose，还基于它，用 shell 开发一个多环境管理版，有趣的，哈哈。

另外，关于是否使用增量迁移方案，具体还是按照项目情况选择吧，可能管理的不仅仅是表结构，有其它更灵活的需求。这种场景下，通过差异打补丁的方式就不适合了。

下篇文章，我将介绍另一个基于差异管理数据表结构的工具，依然是用 Go 实现的。


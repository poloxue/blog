---
title: "一文精通 crontab 从入门到出坑"
date: 2016-12-31T18:01:00+08:00
draft: false
comment: true
tags: ["crontab"]
---

此篇技术博文主要介绍的是 crontab， Linux 下的计划任务管理工具。涉及内容包括 crontab 使用配置、常见坑的分析和编者总结的错误调试方法。

我的理解，后台任务通常分为两种：常驻和定时。之前的文章《pm2进程管理工具使用总结》主要针对的是常驻任务。今天来谈谈 crontab，主要针对的是定时任务。

实验环境： centos7

# 介绍crontab

crontab的服务进程名为crond，英文意为周期任务。顾名思义，crontab在Linux主要用于周期定时任务管理。通常安装操作系统后，默认已启动crond服务。crontab可理解为cron_table，表示cron的任务列表。类似crontab的工具还有at和anacrontab，但具体使用场景不同，可参见附录《让你学会Linux计划任务》一文了解更多。

关于crontab的用途很多，如

- 定时系统检测；
- 定时数据采集；
- 定时日志备份；
- 定时更新数据缓存；
- 定时生成报表；
  ...
  等等任务

当然，更多使用场景是要以视具体情况而定了。毕竟是工具通常都是常用规则总结而成的产物。

确认crond服务已经安装与开启之后，下面开始具体说明

# 简单示例

先来个简单示例体验一下。

- 目标：每分钟向/tmp/time.txt文件下写入当前时间
- 新建crontab任务

```bash
$ crontab -e      // 打开crontab任务编辑
* * * * * date >> /tmp/time.txt
```

- 静静等待几分钟

```bash
$ cat /tmp/time.txt
Do 29. Dez 22:45:01 CST 2016
Do 29. Dez 22:46:01 CST 2016
Do 29. Dez 22:47:01 CST 2016
```
- 从上面结果看出，每分钟执行了date并写入到/tmp/time.txt。

简单示例演示成功。下面从细节深入说明crontab使用。

# 使用选项

上面的实验中使用了crontab命令的-e选项。我们来看看crontab命令中有哪些选项?

**-e 选项** 表示打开当前用户的crontab任务列表配置文件。当然也可以直接打开，路径通常是在/var/spool/cron/下，文件以用户名命名，如/var/spool/cron/root。不过，采用-e方式打开，福利是可以帮助我们自动检查任务配置符合规则。

**-u 选项** 指定某用户的任务列表，很好理解。比如我当前是root用户，想操作poloxue用户的任务列表。如下：

```bash
$ crontab -u poloxue -e
```
**-l 选项** 列出某用户的所有任务列表
**-r 选项** 删除某用户的所有任务列表，这个选项使用小心为上，估计也只是自己实验时玩玩而已，正常不使用。

crontab命令的选项中，主要使用的就是以上几个，理解比较简单。

# 任务配置

说完了crontab的命令选项，下面开始真正的大戏，任务列表文件如何配置？

首先，看下crontab任务列表配置格式，示例文件如下：

```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# 更多细节 man 4 crontabs

# 计划任务定义的例子:
# .---------------- 分 (0 - 59)
# |  .------------- 时 (0 - 23)
# |  |  .---------- 日 (1 - 31)
# |  |  |  .------- 月 (1 - 12)
# |  |  |  |  .---- 星期 (0 - 7) (星期日可为0或7)
# |  |  |  |  |
# *  *  *  *  * 执行的命令
* * * * * date >> /time.txt 2>&1
```
从上面的示例文件可看出，crontab的任务列表主要由两部分组成：环境变量配置与定时任务配置。可能大家在工作中更多是只用到了任务配置部分。

## 环境变量配置部分

理解环境变量配置这部分可以帮助我们减少去踩一些不必要的坑。简单说明上面涉及的环境变量。

**SHELL**为/bin/bash，表示使用/bin/bash解释执行命令

**PATH**表示到哪些目录路径寻找命令程序，此环境变量的值说明了为什么我们在crontab中执行命令时，尽量要写命令全路径才能执行的原因。

**MAILTO**变量作用是当任务执行有输出时，内容发送到哪个用户的邮箱。禁用可以设置MAILTO=""。

当我们在使用crontab时，发现某些定时任务不能顺利执行，但shell控制台执行成功，环境变量是否正确是我们需要首先关注的点之一。具体详情可以看后面关于环境变量坑的说明。

## 定时任务配置部分
这部分是crontab配置核心。

### 基本配置

如下所示配置共6列，前5列是关于执行时间配置，最后1列是具体执行命令。

```bash
.---------------- 分 (0 - 59)
|  .------------- 时 (0 - 23)
|  |  .---------- 日 (1 - 31)
|  |  |  .------- 月 (1 - 12)
|  |  |  |  .---- 星期 (0 - 6) (星期日可为0或7)
|  |  |  |  |
*  *  *  *  * 执行的命令
```
第一列单位为分，表示每时第几分钟，范围为0-59；
第二列单位为时，表示每天第几小时，范围为0-23；
第三列单位为日，表示每月第几天，范围为1-31；
第四列单位为月，表示每年第几月，范围为1-12；
第五列单位为星期，表示每星期第几天，范围0-7，0与7表示星期日，其他分别为星期1-6；

### 时间配置段类型
根据时间列中值的不同设置方式，编者总结出以下五种类型：

**固定某值**，指定固定值，如指定1月1日0时0分执行任务
```bash
0 0 1 1 * command
```
月日时分都指定了固定数值。
<font color=red>注：*在crontab中表示任意值都满足条件。</font>

**列表值**，时间值是一个列表，如指定一个月内2、12、22日零时执行任务
```bash
0 0 2,12,22 * * command
```
上述日指定多个值，2号、12号和22号，以逗号分隔；

**连续范围值**，时间为连续范围的值，如指定每个月1至7号零时执行任务
```bash
0 0 1-7 * * command
```
上述日期为连续范围的值1-7时

**步长值**，根据指定数值跳跃步长确定执行时间，如指定凌晨1时开始每割3个小时0分执行一次任务
```bash
0 1-24/3 * * * command
```
上述指定从凌晨1时每3个小时执行任务，如1点0分，4点0分，7点0分等。

**混合值**，支持以上类型的组合，如指定每小时0至10分，22、33分以及0-60分钟每隔20分钟执行任务，如下
```bash
0-10,22,33,*/20 * * * * command
```
这里的分钟值采取了多种类型组合指定，包括连续范围值(0-7)，列表值(22,33)，步长值(\*/20)。

<font color="red">声明：这几种时间配置类型是编者自己总结，希望能帮助大家更好理解。有错误帮忙指出。</font>

## 定时语句解析工具

通常在使用crontab添加任务时，我们会依靠自己已有知识编写定时语句。当需要测试语句是否正确时，总需要一定时间等待证明其正确性。作为一名牛逼的程序员，这种方式就太不酷了。有没有一款工具，只要我们给出语句，其就能告诉具体执行时间呢？下面介绍一款老外开发的crontab在线解析工具。

工具地址：https://crontab.guru

下面是这个工具的截图

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2016-12-31-crontab-from-scratch-01.png)

从上面看出，我们输入的语句解析结果为每天的04：05执行任务。下面有这样一行文字“next at 2016-12-31 04:05:00”，告诉了我们最近一次的执行时间。

<font color=red>注明：百度搜索“crontab在线解析”获得的工具有坑，某些语句解析结果错误。为避免大家受骗，这里提供具体地址：http://tool.lu/crontab/</font>

# 使用有坑

crontab使用中常会遇到各种坑。下面列出编者在使用中曾遇到的一些问题。

## 时间配置误区

此处介绍两种坑，一种是由于基本功不足导致配置错误，而另一种则是多数人对crontab配置都存在的一个理解误区。

## 整点时间设置错误

其实这个错误不用单独说明，但是编者刚开始接触crontab时犯过，单独拿出来说明一下。

如设定每天3点执行一次某任务

下面列出错误方式，当我们听到每天3点执行一次某任务时，很多人会把重点放在3点，而忽略了执行一次的需求。

下面是个错误的例子
```
* 3 * * * command
```

这里会导致在三点的每分钟都会执行一次任务，也就是执行了60次。
正确方式如下，每天3点0时执行任务

```bash
0 3 * * * command
```

## 日与星期的关系误区
这真的是个大误区，很多人都不知道的大误区。直接开始说明吧。

好，首先做两个练习

设置任务一：每月的1-7每天零时执行某任务，答案如下：
```bash
0 0 1-7 * * date >> /tmp/date.txt
```
设置任务二：每星期的星期一零时执行某任务，答案如下：
```bash
0 0 * * 1 date >> /tmp/date.txt
```

上面两个任务的设定都是正确的。

下面提出第三个任务，设置每个月的第一个星期一零时执行某任务

分解任务要求，首先，第一个星期就是每个月的1-7日，而星期一就是星期一。所以我们理解的crontab任务配置如下

```bash
0 0 1-7 * 1 date >> /tmp/date.txt
```
下面直接使用前面介绍的在线解析工具分析此语句，如下

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2016-12-31-crontab-from-scratch-02.png)

解析结果显示语句执行时间为每月的1至7日和每星期一。可以看到最近执行时间是“next at 2017-01-01 00:00:00”，这个时间也并非星期一。

这是crontab的一个特别容易误解之处，下面直接给出结论:

- 当日和星期任一列包含*时，日与星期两者为并且的关系；
- 当日和星期列中不包含*时，日与星期两者为或者的关系；

<font color=red>请注意，前面提到的那个百度搜索出来的工具分析结果显示的确是每月第一个星期一，这是错误的。如有朋友持怀疑态度，可自行验证，如有错误，随时告知。</font>

## 环境变量问题

当我们刚使用crontab时，有人会告知所有命令尽量都使用绝对路径，以防错误。为什么？这就和我们下面要谈的环境变量有关了。

首先，获取控制台环境变量看下

```bash
$ env
XDG_SESSION_ID=10
HOSTNAME=localhost.localdomain
SHELL=/bin/bash
PERL_MB_OPT=--install_base /root/perl5
USER=root
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/php5/bin
PWD=/var/mail
SHLVL=1
HOME=/root
LOGNAME=root
XDG_RUNTIME_DIR=/run/user/0
_=/usr/bin/env
```
<font color=red>考虑篇幅，输出有删减。</font>

然后，获取crontab环境变量信息
```bash
* * * * * /usr/bin/env > /tmp/env.txt
```

输出结果，如下
```bash
$ cat /tmp/env.txt
XDG_SESSION_ID=732
SHELL=/bin/sh
USER=root
PATH=/usr/bin:/bin
PWD=/root
LANG=de_DE.UTF-8
SHLVL=1
HOME=/root
LOGNAME=root
XDG_RUNTIME_DIR=/run/user/0
_=/usr/bin/en
```

对比分析两者输出

对比crontab与控制台输出，我们发现两者的环境变量差异很大。如果命令在控制台执行成功，而在crontab执行失败，我们需要考虑是否命令涉及的环境变量在crontab和控制台间存在差异。

明白crontab使用绝对路径执行命令原因了吗？

我们知道命令默认查找路径是由PATH指定的。

从上面输出结果可知，控制台的PATH值为
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/php/bin
```

crontab的PATH值为
```bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/php/bin
```

crontab的PATH值为
```bash
PATH=/usr/bin:/bin
```

/usr/local/php/bin/下面存在php命令，在控制台执行成功
```bash
$ php index.php
```

因在crontab的PATH变量无/usr/local/php/bin/，其执行php命令则会失败。

## 解决方式

已知哪个环境变量导致问题，可以直接在crontab配置中加入变量配置。

不知哪个环境变量导致问题，终极大招是引入控制台环境变量，如下

```bash
* * * * * source /$HOME/.bash_profile && command
```

当然，对于某特定环境变量或有特定的处理方式，如PATH，命令使用绝对路径亦可解决。

## 特殊符号%

%在crontab是特殊符号，具体含义如下：

第一个%表示标准输入的开始

```bash
* * * * * cat >> /tmp/cat.txt 2>&1 % stdin input
```

执行成功之后，查看/tmp/cat.txt

```bash
$ cat /tmp/cat.txt
stdin input
```

我们看到标准输入写入到了/tmp/cat.txt文件。

<font color=red>理解上面示例，首先需知cat >> /tmp/cat.txt ，作用是将标准输入重定向至/tmp/cat.txt。</font>

其余%表示换行符

示例如下
```bash
* * * * * cat >> /tmp/cat_line.txt 2>&1 % stdin input 1 % stdin input 2 % stdin input 3
```

查看输出
```bash
$ cat /tmp/cat_line.txt
stdin input 1
stdin input 2
stdin input 3
```

有三行输出

解决方式

既然是特殊字符，自然而然就想到了使用\进行转义，如下：
```bash
* * * * * cat >> /tmp/cat_special.txt 2>&1 % per cent is \%. 2>&1
```

查看输出
```bash
$ cat /tmp/cat_special.txt
per cent is %.
```

执行成功了。自此，你就顺利爬出了%特殊字符问题的坑。

关于这个问题的具体说明，可以参看附录中的《Crontab and %》。

## 关于输出重定向

当我们不做输出重定向时，如任务有大量输出，或许有些无法解释的问题。

输出写入邮件

crontab任务输出默认写入到执行用户的邮件中，如下演示：
```
* * * * * date
```

命令输出当前日期，下面查看当前用户的邮件
```bash
$ cat /var/spool/mail/$USER
...

Sat Dec 31 17:45:01 CST 2016
```
由此可见，任务输出的日期信息写入到了用户邮件中。

如任务有大量输出，会占用磁盘资源。但编者测试显示，如磁盘容量不足，任务也会执行，但输出不会写入邮件；

关闭邮件功能

如何关闭？设置MAILTO环境变量为空。如下
```bash
MAILTO=""
* * * * * date
```

是不是关闭邮件写入就好了？附录《Linux中的crontab与sendmail》博文表明，关闭mail功能，输出内容将写入到/var/spool/clientmqueue中，可能占满分区的inode资源，导致任务无法执行。inode资源使用情况可通过如下命令获取
```bash
$ df -i
Filesystem Inodes   IUsed  IFree    IUse% Mounted on
/dev/sda1  512000   378    511622   1%    /boot
/dev/sda2  92672000 185351 92486649 1%    /
```

抱歉！这种情况编者并未测出！但在公司的生产环境发现过未重定向则任务不执行的情况，加上后解决了问题。百度也搜索到了类似问题，如有朋友了解，欢迎指教，万分感谢。

当然，为了避免此类问题发生，建议任务都加上输出重定向，如下

```bash
* * * * * date >> /dev/null/ 2>&1
```
输出到/dev/null中，标准输入和标准错误都应处理。

如大家对重定向有疑惑，可参见附录中的《Linux重定向》，对文解释不错。

<font color=red>程序员的感悟：在技术的世界，当我们不按常理做事，事情也不会按常理犯错。</font>

# 调试大招

最后的福利，编者根据自己的总结而梳理出一套快速定位crontab错误的思路。两个角度：

- 任务是否执行
- 命令是否正确

## 任务是否执行？

调试思路

首先，通过日志确认任务是否执行
然后，如未执行则分析定时语句，
最后，定时没有问题，检查crond服务是否开启

下面说明具体分析步骤。

日志确认

调试错误，日志通常是个利器，crontab也有日志。
编者的服务器中crontab日志文件位置为/var/log/cron

**查看日志**
日志中包含任务执行记录，配置错误提示，任务配置编辑重载记录，服务开启等记录。

下面是日志的部分内容，
```bash
$ vim /var/log/cron
...
Dec 31 19:17:01 localhost crond[1455]: (CRON) bad day-of-week (/var/spool/cron/root)
Dec 31 19:17:01 localhost CROND[4409]: (root) CMD (date)
...
```

这里截取了对调试比较重要的两条记录，如下介绍

**执行记录**

```bash
Dec 31 19:17:01 localhost CROND[4409]: (root) CMD (date)
```
显示12月21 19时17分1秒执行了date命令

**配置错误**

```bash
Dec 31 19:17:01 localhost crond[1455]: (CRON) bad day-of-week (/var/spool/cron/root)
```
上面显示/var/spool/cron/root的任务配置有错，也就是root任务配置有错。错误原因：bad day-of-week，星期配置有错。

语句是这样的
```bash
* * * * date >> /dev/null 2>&1
```

明显缺少了星期时间段。

确认定时语句

通过上面的日志分析，如任务没有执行，使用定时语句在线分析工具分析定时是否正确，非常简单。

确认服务开启
如果定时语句也正确，检查服务是否开启。检测命令如下

Systemd方式(centos7及以上)
```bash
$ systemctl status crond.service
```
SysVinit方式(centos7以下)
```bash
$ service crond status
```
查看命令输出，如未开启，执行如下命令开启

Systemd方式(centos7及以上)
```bash
$ systemctl start crond.service
```
SysVinit方式(centos7以下)
```bash
$ service crond start
```
确认任务成功后，如问题仍未解决，继续往下看。

## 命令是否正确

确认命令成功与否，这里总结步骤大致如下

获取命令执行输出

crontab中的命令执行出错，多数人都不知道如何调试。我们知道在控制台执行命令时，可通过输出获取错误信息调试问题。这种方式在crontab同样适用，方法就是利用重新向获取输出，进行分析。示例如下

```bash
* * * * * php /root/index.php >> /tmp/debug.log 2>&1
```

这条任务总是执行失败，我们把输出重定向到/tmp/debug.log。

查看debug.log，如下

```bash
$ cat /tmp/debug.log
/bin/sh: php: command not found
/bin/sh: php: command not found
```

显示php命令没有找到，很明显的就可以确定是环境变量的问题。这种方式定位问题非常有效。

具体问题具体分析

有了命令执行的输出，下面就是具体问题具体分析了。或许是前面提到的各种坑，也或许是命令本身所独有的问题。

调试的方法到这里就说完了。但还是实践为王，需持续总结，同时也希望大家不要在同样的坑中重复犯错。

crontab写了这么长，希望能切实帮到大家。有哪位朋友看到了最后吗？表示佩服！

# 参考附录

- [让你学会Linux计划任务](http://os.51cto.com/art/201001/176402.htm)
- [Linux中的crontab与sendmail](http://www.server110.com/sendmail/201311/3125.html)
- [Crontab and %](http://www.hcidata.info/crontab.htm)
- [Linux重定向](http://www.jb51.net/os/RedHat/1120.html)

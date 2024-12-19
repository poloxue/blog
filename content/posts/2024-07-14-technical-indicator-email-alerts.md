---
title: "Python 实现技术指标邮件告警通知"
date: 2024-07-14T18:53:16+08:00
draft: false
comment: true
description: "在交易中，如果你有一套人工配合机器人的交易思路，即希望人工确认而不直接下单，那么及时获取关键位置的通知信息，如一些特征明显的技术指标的告警信息，是非常重要的。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-07/2024-07-14-technical-indicator-email-alerts-00.webp)

在交易中，如果你有一套人工配合机器人的交易思路，即希望人工确认而不直接下单，那么及时获取关键位置的通知信息，如一些特征明显的技术指标的告警信息，是很有帮助的。

市面上有不少的付费软件支持这个能力，如果你可以使用 TradingView，它的免费版也可以做到，缺点是缺少国内期货品种。我是不想花钱的，秉承着自己动手，丰衣足食，毕竟小散户不是大机构。

本文将基于开源数据方案，通过 Python 一套免费的方案，编写指标告警并将其发送到邮件。我将利用 akshare 获取历史行情数据，使用 TA-Lib 计算技术指标。

接下来将按照这个流程逐步展开介绍。 

## 数据下载

在上一篇文章中，我们介绍了如何使用 akshare 下载国内期货、股票和指数的历史行情。假设，我们已经下载了数据，将其存储在一个 DataFrame 中。

如下是上证指数 000001 的数据下载，我们可直接使用上篇文章的 history_bars 方法，并将数据保存到 `index_000001.csv` 文件中。

```python
# 下载上证指数的历史行情数据
df = history_bars(symbol="000001", length=100)
df.to_csv("index_000001.csv", index=False)
```

提前下载行情是为了防止重复调用产生可能的网络问题，把行情查询和指标计算分离，提高报警程序稳定性。

## 安装 TA-Lib 库

现在可以开始计算指标了。

python 的指标计算库有如 talib、pandas-ta、ta 等等，其中 talib 最出名，支持超过 150 中技术指标，由 C 语言实现，性能更高。我将使用的就是 talib。

TA-Lib 的安装要提前安装依赖的动态库，参考 [TA-Lib 官方文档](https://ta-lib.github.io/ta-lib-python/install.html)。我简单展开说说不同操作系统下依赖库的安装方法吧。

**macOS**

macOS 上可通过 Homebrew 安装：

```bash
brew install ta-lib
```

**Linux**

Linux 上要手动编译安装，提前下载 [ta-lib-0.4.0-src.tar.gz](https://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz)，并执行如下命令：

```bash
$ wget https://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz
$ tar zxvf ta-lib-0.4.0-src.tar.gz
$ cd ta-lib/
$ ./configure --prefix=/usr
$ make
$ sudo make install
```

**Window**

Windows 上下载 [ta-lib-0.4.0-msvc.zip](https://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-msvc.zip) 文件，并将其解压到 `C:\ta-lib`。

我记得 windows 上也是可以直接下载 wheel 文件安装 ta-lib，我很早之前使用 windows 就是通过这种方式安装的。

**pip install**

在依赖库安装完成后，直接 pip 即可安装 TA-Lib。

```bash
pip install ta-lib
```

到此，TA-Lib 就安装成功了。有不少人开始入手指标计算的时候，都被 TA-Lib 的安装给卡住了，希望以上的介绍对你有用。

## 指标计算与告警

我们开始计算指标和告警规则，我会用 RSI 和布林带这两个耳熟能详的指标，基于它们配置一些报警规则。

首先加载 `index_000001.csv` 中保存的数据，进行清洗和预处理，便于指标计算。

```python
df = pd.read_csv("index_000001.csv")
df['date'] = pd.to_datetime(df['date'])
df.set_index('date', inplace=True)
```

接下来，使用 TA-Lib 计算 RSI 和布林带（BollingerBands）。

```python
import talib

# 计算 RSI
df['RSI'] = talib.RSI(df['close'], timeperiod=14)

# 计算布林带
df['upperband'], df['middleband'], df['lowerband'] = talib.BBANDS(df['close'], timeperiod=20)
```

如果想了解更多技术指标的计算方法，可直接查看 TA-Lib 的文档，它还可以计算其他常见的技术指标，例如 MACD 和移动平均线。

```python
# 计算 MACD
df['macd'], df['macdsignal'], df['macdhist'] = talib.MACD(df['close'])

# 计算移动平均线
df['SMA_50'] = talib.SMA(df['close'], timeperiod=50)
df['SMA_200'] = talib.SMA(df['close'], timeperiod=200)
```

基本在市面上常见的指数指标在 TA-Lib 中都可以找到，查看它的 [指标函数列表](https://ta-lib.github.io/ta-lib-python/funcs.html)。

有了指标后，开始定义报警条件。我为了演示，定义 RSI 和布林带的告警条件为常用的超买信号，即 RSI 大于 70 和收盘价大于布林带上轨为报警条件。

基于如下代码：

```python
rsi_alerts = df['RSI'] > 70  # RSI 超过 70
bollinger_alerts = df['close'] > df['upperband']  # 收盘价超过上轨
```

有了两条告警后，设置为每天收盘后运行，决定第二天是否交易。

或许，你希望收盘临近时检测，从而在收盘前提前入场，不过这个前提是要能获取到实时行情，akshare 中也提供了这样的接口，可查看它的 [接口列表](https://akshare.akfamily.xyz/tutorial.html) 查找。

## 发送告警信息

告警规则有了，将信息发送到我们就变得非常重要了，我选择的发送到邮件。我将其封装消息发送函数。

```python
send_email(title, body, to_email)
```

使用 Python 的 smtplib 库可以轻松发送邮件。当然，你需要提前开始 smtp 支持，如 163 邮箱，登录到邮箱的设置页 -> POP3/SMTP/IMAP，启用 POP3/SMTP 服务，会得到一串授权码，而 163 的 smtp 的地址和 SSL 的端口分别是smtp.163.com 和 465。

```python
import smtplib
from email.mime.text import MIMEText

def send_email(subject, body, to_email):
    from_email = 'your_email@163.com'
    from_password = 'your_email_password' # smtp 的授权码

    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = from_email
    msg['To'] = to_email

    server = smtplib.SMTP_SSL('smtp.163.com', 465)
    server.login(from_email, from_password)
    server.sendmail(from_email, [to_email], msg.as_string())
    server.quit()
```

只要一行代码即可测试，如下：


```python
send_email("你好", "测试邮件", "to_email@163.com")
```

万事具备，我们现在可以发送告警信息了。

```python
alerts = rsi_alerts | bollinger_alerts
if alerts.iloc[-1] < 50:
    last_close = df["close"].iloc[-1]
    last_rsi = df["RSI"].iloc[-1]
    last_upperband = df["upperband"].iloc[-1]
    message = f"告警！RSI：{last_rsi}，收盘价：{last_close}，上轨：{last_upperband}"
    send_email("告警！", message, "poloxue123@gmail.com")
```

实现的逻辑还是非常简单的，当然告警的条件要视你的交易思路来实现。

### 示例项目

以下是一个完整代码，将上面的过程整合到了一起、指标计算到发送告警信息的全过程：

```python
import pandas as pd
import talib

import smtplib
from email.mime.text import MIMEText

df = pd.read_csv("index_000001.csv")
df["date"] = pd.to_datetime(df["date"])
df.set_index("date", inplace=True)

# 计算 RSI
df["RSI"] = talib.RSI(df["close"], timeperiod=14)

# 计算布林带
df["upperband"], df["middleband"], df["lowerband"] = talib.BBANDS(
    df["close"], timeperiod=20
)

# 计算 MACD
# df['macd'], df['macdsignal'], df['macdhist'] = talib.MACD(df['close'])

# 计算移动平均线
# df['SMA_50'] = talib.SMA(df['close'], timeperiod=50)
# df['SMA_200'] = talib.SMA(df['close'], timeperiod=200)

rsi_alerts = df["RSI"] > 70  # RSI 超过 70
bollinger_alerts = df["close"] > df["upperband"]  # 收盘价超过上轨


PASSWORD = "your_email_password"


def send_email(subject, body, to_email):
    from_email = "your_email@163.com"
    from_password = PASSWORD

    msg = MIMEText(body)
    msg["Subject"] = subject
    msg["From"] = from_email
    msg["To"] = to_email

    server = smtplib.SMTP_SSL("smtp.163.com", 465)
    server.login(from_email, from_password)
    server.sendmail(from_email, [to_email], msg.as_string())
    server.quit()


alerts = rsi_alerts | bollinger_alerts
if alerts.iloc[-1] < 50:
    last_close = df["close"].iloc[-1]
    last_rsi = df["RSI"].iloc[-1]
    last_upperband = df["upperband"].iloc[-1]
    message = f"告警！RSI：{last_rsi}，收盘价：{last_close}，上轨：{last_upperband}"
    send_email("告警！", message, "to_email@163.com")
```

注意将以上的邮件地址和授权码替换为真实的即可。

## 定时执行

现在，我们只要配置脚本的定时执行即可，可以使用 python 的 schedule 包，或是在 Linux
 系统上，直接通过 Crontab 配置定时任务。

演示案例：如配置周一到周五，每天9点检查并告警。

如果使用 schedule，要讲如上的过程封装下便于调用。

```python
import time
import datetime
import schedule


def indicator_alerts():
  # 你的检测代码
  pass

# 配置周一到周五的早上九点执行任务
schedule.every().monday.at("09:00").do(indicator_alerts)
schedule.every().tuesday.at("09:00").do(indicator_alerts)
schedule.every().wednesday.at("09:00").do(indicator_alerts)
schedule.every().thursday.at("09:00").do(indicator_alerts)
schedule.every().friday.at("09:00").do(indicator_alerts)

while True:
    schedule.run_pending()
    time.sleep(1)
```

如果是 Crontab，直接配置定时任务即可，如下所示：

```python
0 9 * * 1-5 python your_script.py
```

现在，我们就可以实现对基于技术指标的告警配置，还能通过邮件接收告警信息。

希望这篇文章对你有所帮助。如果有任何问题或建议，欢迎留言讨论。



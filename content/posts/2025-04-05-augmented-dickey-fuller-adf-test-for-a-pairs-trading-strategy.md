---
title: "配对交易策略中的 ADF Test"
date: 2025-04-04T15:29:53+08:00
draft: true
comment: true
description: "本文是配对策略中的 ADF Test 教程。"
---

翻译：[Augmented Dickey Fuller (ADF) Test for a Pairs Trading Strategy](https://blog.quantinsti.com/augmented-dickey-fuller-adf-test-for-a-pairs-trading-strategy/)

Augmented Dickey Fuller Test, denoted by "ADF", serves the purpose of finding out which stocks can be paired together in the pairs trading strategy. For the strategy to work effectively, it is important to find the pair of stocks that are co-integrated.

The statistical calculation for testing the co-integration is done with Augmented Dickey Fuller Test. In this blog, we will be discussing the ADF test for pairs trading strategy.

## What is a pairs trading strategy?

The origin of the pairs trading strategy was back in the early 1980s. The strategy was initially developed by stock market technical analysts and researchers who were employees of the renowned investment banking and financial company – Morgan Stanley.

In order to implement the pairs trading strategy, one needs technical and statistical analysis skills. The pairs trading strategy basically requires two stocks or securities to co-integrate so that they can be considered as a pair for trading.

Hence, a positive correlation is needed.

After the securities show co-integration, from the pair of stocks, you enter a long position in one stock and a short position in the other. The positions are based on the current market price of both the stocks and their standard deviation.

## Essential terms used in the Augmented Dickey Fuller Test

### Stationary time series

Stationarity means that the statistical properties of a time series i.e. mean, variance and covariance do not change over time which implies that the time series has no unit root.

In the case of stationarity, trading signals are generated assuming that the prices of both stocks will revert to the mean eventually. Hence, we can take the advantage of the prices that deviate from the mean for a short period of time.

Grasping the concept of Stationary Time Series can significantly enhance your trading strategy. By understanding how mean reversion works in stationary data, you can identify profitable opportunities when prices temporarily deviate. Explore more to discover how this powerful concept can improve your decision-making and increase the effectiveness of your trading signals.

### Non-stationary time series

A time series with a unit root is non-stationary and will have changes in its mean, variance and covariance over time. Due to the non-stationarity of time series, the trading signals cannot be generated.

Below, you can see a plot of an asset which is a non-stationary time series with a deterministic trend represented by the “blue” curve and its detrended stationary time series represented by the “red” curve.

### Unit root

The unit root is a characteristic of a time series that makes it non-stationary. Technically speaking, a unit root is said to exist in a time series of the value of alpha = 1 in the below equation.

$$Y_t = {\alpha}yt_1 + {\beta}X_e + \epsilon$$

where, Yt is the value of the time series at the time ‘t’ and Xe is an exogenous variable (a separate explanatory variable, which is also a time series).

## What is ADF testing?

The Augmented Dickey Fuller test (ADF) is a modification of the Dickey-Fuller (DF) unit root.

Dickey-Fuller used a combination of T-statistics and F-statistics to detect the presence of a unit root in time series.

ADF test in pairs trading is done to check the co-integration between two stocks (presence of unit root).

If there is a unit root present in the time series, it implies that the time series is non-stationary and the stocks are not co-integrated. Hence, stocks cannot be traded together.



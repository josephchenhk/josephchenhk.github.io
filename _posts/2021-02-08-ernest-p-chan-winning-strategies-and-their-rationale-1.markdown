---
title: 'Ernest P. Chan: Winning Strategies and Their Rationale (1)'
layout: post
date: 2021-02-08 16:26
image: "/assets/images/markdown.jpg"
headerImage: false
tag:
- 算法交易
- qtrader
star: true
category: blog
author: joseph
description: 读书笔记
---

A brief note of Ernest Chan's book. 

# Backtesting and Automated Execution

* backtest要做对，否则不如不做

* 好的回测平台 (backtesting platform)，应该与好的自动化执行平台 (automated execution platform)相一致 (容易从回测过渡至实盘交易)

## The Importance of Backtesting

为什么别人的策略，到你这就很难重复 (backtest a published strategy)?

* 别人的细节不会告诉你，只有自己亲自backtest一遍，踩了所有的坑，才算pick up了别人的strategy

* 自己实现backtesting，能够让你将别人的策略，跑在真正的out-of-sample的数据集上；如果不work，你很可能可以判断这是一篇水文

* 自己实现backtesting，也是一个改进策略 (refine and improve the strategy)的过程

## Common Pitfalls of Backtesting

### Look-ahead Bias

任何时候，一个正确的backtesting，都不应该用到未来的数据

### Data-Snooping Bias and the Beauty of Linearity

Data-snooping：用太多参数去过拟合历史数据。怎么解决？

* 严格执行out-of-sample一票否决权（另一种不那么好的方法是：进行cross validation）

* 尽可能减少参数（parameter）。【注意】如果你的策略参数不多，但有很复杂的交易规则，也可能导致data-snooping

### Linear Model or Nonlinear Model

不要迷信非线性模型，很多时候简单的线性模型，反而更好

### Stock Splits and Dividend Adjustments

* 拆股N-to-1，股票价格会被稀释N倍：回测时将除权日前的价格除以N（在实盘时也要做类似的处理）

* 派息每股¥d，股票价格会降低¥d：回测时将除权日前的价格减去¥d（在实盘时也要做类似的处理）

自己做一次前复权/后复权

### Survivorship Bias in Stock Database

如果回测的数据里，不包括除牌（delisted）的公司，就存在survivorship bias了。对于mean-reverting long-only的策略而言，这种幸存者偏差的影响尤其重要；对momentum策略而言，影响相对没那么大。

mean-reverting策略更依赖长期的数据，filter掉退市的股票，变相等于避免了一部分可能不盈利的策略，从而导致回测结果inflated；momentum策略更依赖短期的数据，filter掉退市的股票，变相等于放弃了一部分可能短期内盈利的股票，从而导致回测结果deflated。

### Primary versus Consolidated Stock Prices

一只股票在多个交易所上市，在回测时要留意你用到的是哪个交易所的数据。如果使用consolidated prices做回测，很容易搞不清楚实际交易时到底应该用哪个交易所的价格。

Bar data某程度上就是一种consolidated data，使用OHLC数据进行回测时，要留意与真实的price feed的区别。

Tick data可以通过订阅实时数据，自行进行收录（如果技术上做得到的话）。

### Venue Dependence of Currency Quotes

外汇市场，不同的venue有不同的报价，如果我们打算在A venue进行交易，那就必须用A venue的历史数据进行回测；如果用B venue的历史数据，那就是一个伪回测，
不具备实际意义。

如果用quote aggregator做回测的historical data，将来打算交易的venue，一定要在这个aggregator里面，否则也是一个无效的回测。

外汇市场数据还有一个特点：通常bid/ask quotes数据相对容易获得，但trade price/volume较难获得（因为没有监管要求上报trade数据，大家都会倾向将trade
数据看作是自己的私有财产）

## Short-Sale Constraints

通常，你的经纪商（broker）未必支持short sale。

如果short interest足够多，会导致borrow share的成本很高。

* "uptick rule"：short sale的价格要比最近成交价更高。

## Futures Continuous Contracts

期货需要考虑roll over

## Futures Close versus Settlement Prices

Settlement prices由exchange提供，即使当天没有交易，也会有一个settlement prices。用actural transaction prices去做回测比较好，如果用
settlement prices，要小心。

## Statistical Significance of Backtesting: Hypothesis Testing


## Choosing a Backtesting and Automated Execution Platform

买一个系统还是自己build一个系统？

## Can Backtesting and Execution Use the Same Program?

应该要一致：

（1）方便将backtest strategy 迁移至实盘live trading（尽可能少的改动代码）

（2）避免look-ahead bias

## What Type of Asset Classes or Strategies Does the Platform Support?

市面上已有的平台，通常只能trade一种或几种资产。**自己写会有足够的灵活性** （例如[Qtrader](https://github.com/josephchenhk/qtrader)）。


## Does the Platform Have Complex Event Processing?

> For an algorithmic trader, one important point is that the program is event driven, and not bar driven.That is, the program does not go poll prices or news items at the end of each bar and then decide what to do. Because CEP is event driven, there is no delay between the occurrence of an event and the response to it.

Jospeh：这未必是一条金科玉律。很多中低频的systematic trading strategies其实也是由bar驱动的。

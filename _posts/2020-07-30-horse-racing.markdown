---
title: " 机器学习预测赛马（一）"
layout: post
date: 2020-07-30 14:26
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Machine learning
- horse racing
- hkjc
- Neural network
star: true
category: blog
author: joseph
description: Use machine learning techniques to predict horse racing
---
 
# 预测什么
赛马预测是一个较少被公开研究的话题（成功的话早就闷声发大财了），这里做一些总结。首先，我们要预测什么？其实，以下几种预测都是常见的：

1. 预测每匹马的完成时间

2. 预测每匹马是“赢”还是“输”（binary classification）

3. 预测每匹马的最终排名（multi-class classification）

其中，第2和第3种预测都是各自明显的缺陷：对于二元分类，一个显著的问题是，赛马的数据是imbalanced的，即赢的马匹是少数，输的是大多数；对于多元分类，则会有难以解释的地方，比如对一场比赛的排名预测如下：

| Horse\Place | 1st | 2nd | 3rd |
|:------------|:---:|:---:|:---:|
|  #1         | 60% | 40% | 20% |
|  #2         | 30% | 60% | 50% |
|  #3         | 50% | 40% | 60% |

---
title: 'Ernest P. Chan: Winning Strategies and Their Rationale (3)'
layout: post
date: '2021-04-01 09:52:38'
image: assets/images/markdown.jpg
headerImage: false
tag:
- 算法交易
star: true
category: blog
author: joseph
description: 读书笔记
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# Tests for Time Series Momentum

计算时间序列的correlation，首先需要**选取一个time lag**，站在某个时间点上，我们总会有两个return：过去一段时间(-N)的return，和未来一段时间(+M)的return，我们需要计算在时间序列上两者的相关系数：

$$Corr(R(-N, t), R(+M, t)) = \rho(R(-N, t), R(+M, t)) $$

尽可能尝试不同的(N,M)组合，去找出最optimal（相关系数最大）的look-back period (N) 和holding period (M).

计算时，需要注意要么look-back period不能重叠，要么holding period不能重叠，如下图所示：

![Markdown Image][1]

# Hurst exponent together with the Variance Ratio test
“variance-ratio test”又称作 “F-ratio test” 或者F-test.  F-test 主要表征两个分布的方差的比例

$$F = \frac{s_1^2}{s_2^2}$$

其中，$s_1^2=\frac{\sum(x_1-\bar{x}_1)^2}{n_1-1}$， $s_2^2=\frac{\sum(x_2-\bar{x}_2)^2}{n_2-1}$


[1]: /assets/images_in_posts/overlap_lookback_holding.png

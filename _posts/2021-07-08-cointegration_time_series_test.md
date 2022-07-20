---
title: Cointegration Time Series Tests
layout: post
date: '2021-09-28 09:55:00'
image: assets/images/markdown.jpg
headerImage: false
tag:
- systematic trading
- statistics
star: true
category: blog
author: joseph
description: knowledge of statistics
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


In a previous [post](https://josephchenhk.github.io/stationary_time_series_test/), we examined the fundamental tools to test for stationarity on time series using Python. If we use the tools described in the article, we will very soon realise that most time series are neither stationary nor mean reverting. In this new article we are going to examine how we can test two (or more) non-stationary time series to check whether the combined value is stationary.

This is where we should introduce the notion of $\color{blue}{\textbf{cointegration}}$ .

`If we are able to find a stationary linear combination of several time series that are not themselves stationary, then these are called cointegrated.`

We are going to see two different tests: the simpler **Cointegrated Augmented Dickey-Fuller test (CADF)** will be useful for pairs only, but we can apply the **Johansen test** to any number of time series.

# Cointegrated Augmented Dickey-Fuller Test
In the previous post, we saw how the ADF and Variance Ratio can test a given time series for mean reversion and stationarity, but we don’t know the number of units o percentage we should use to combine them into the stationary basket of elements we are looking for.

`We have to be aware that just because a set of time series is cointegrating doesn’t mean that any random linear combination of the series will form a stationary basket of elements.`

To easily create the test we can use the procedure by Engle and Granger, which can be defined as the following steps:

1. Determine the optimal hedge ratio by running a lineal regression fit between the two series.
2. Use the hedge computed in step 1 to form a portfolio.
3. Run a stationarity test on the portfolio created in step 2.

Taking advantage of the code already written in the previous article, we can write the test easily with the help of the numpy and statsmodels libraries in the following way:

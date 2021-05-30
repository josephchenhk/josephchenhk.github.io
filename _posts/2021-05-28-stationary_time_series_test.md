---
title: Stationary Time Series Tests
layout: post
date: '2021-05-28 09:55:00'
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


In the previous [post](https://josephchenhk.github.io/sample-mean-and-variance/), we investigated the population mean and variance, and the sample mean and variance. We also understand what is an unbiased estimator of the population variance. Now we are going to look for ways to test for mean reversion on time series using the Python programming language, which will give us the basic toolbox to deal with cointegration in future posts.

# Mean Reversion Process
Mathematically, a continuous mean-reverting time series can be represented by an Ornstein-Uhlenbeck stochastic differential equation in the following form:

$$
\begin{equation} \label{ou-eq}
dx_t = \theta(\mu - x_t)dt + \sigma dW_t
\end{equation}
$$

Where $\theta$ is the rate of reversion to the mean, $\mu$ is the mean value of the process, $\sigma$ is the standard deviation (and $\sigma^2$ is the variance) of the process and, finally, $W_t$ is a Wiener process. The given equation implies that the change of the time series in the next period is proportional to the difference between the mean and the current value, with the addition of Gaussian noise.

In discrete form, this OU equation is given as (just let $\Delta t=1$, and change differentiaon to $\Delta$)

$$
\begin{equation} 
\Delta x_t = \theta(\mu - x_{t-1}) + \sigma \Delta W_t
\end{equation}
$$

where $\Delta x_t = x_t - x_{t-1}$.
# Augmented Dickey-Fuller Test
The Augmented Dickey-Fuller test provides a quick check and confirmatory evidence that your time series is stationary or non-stationary. We should take into account that is a statistical test, and as such it can only be used to inform the degree to which a null hypothesis can be accepted or rejected. So the given result must be interpreted to be meaningful.

The ADF test is based on the simple observation that if the value level is higher than the mean, the next move will be downward while if the value is lower than the mean, the next move will be upward.

We can describe the value changes with the following linear model:

$$
\begin{equation} 
\Delta x_t = \lambda x_{t-1} + \mu + \beta t + \alpha_1 \Delta x_{t-1} + ... + \alpha_k \Delta x_{t-k}  + \epsilon_t
\end{equation}
$$

where $\Delta x(t) = x(t) - x(t-1)$, $\Delta x(t-1) = x(t-1) - x(t-2)$ and so on.

The function of the ADF is to test if $\lambda = 0$. **If the hypothesis $\lambda = 0$ can be rejected**, that means the next move $\Delta x(t)$ depends on the current level $x(t-1)$ and so **it is not a random walk**. The test statistic is the regression coeficient $\lambda$ divided by the standard error of the regression fit:

$$
\begin{equation} 
DF = \frac{\lambda }{SE(\lambda)}
\end{equation}
$$

Dickey and Fuller have already calculated the distribution of this test statistic, which allows us to determine the rejection of the hypothesis for any chosen percentage critical value. Since we expect mean regression to be negative (i.e., $\lambda$ is expected to be negative here if this is a mean-reverting time series) and it has to be more negative than the critical value for the hypothesis to be rejected.

We can simply interpret the result using the p-value from the test. A p-value below a specified threshold (we are going to use 5%) suggests we reject the null hypothesis (stationary), otherwise a p-value above the threshold suggests we accept the null hypothesis (non-stationary).

A naive implementation of the ADF test in Python is shown here:

```
import numpy as np
from statsmodels.regression.linear_model import OLS
from statsmodels.tsa.tsatools import lagmat, add_trend
from statsmodels.tsa.adfvalues import mackinnonp

def adf(ts, maxlag:int=None):
    """
    Augmented Dickey-Fuller unit root test

    tsdall[explanatory variable] (maxlag=3 as an example)
    * 1st column(tsdall[:,0]): [x3, x4, ..., x_{t-2}, x_{t-1}] are X variables (corresponding to coefficient *lambda*)
    * 2nd column(tsdall[:,1]): [x3-x2, x4-x3, ..., x_{t-2}-x_{t-3}, x_{t-1}-x_{t-2}] are X variables (corresponding to coeff *alpha_1*)
    * 3rd column(tsdall[:,2]): [x2-x1, x3-x2, ..., x_{t-3}-x_{t-4}, x_{t-2}-x_{t-3}] are X variables (corresponding to coeff *alpha_2*)
    * 4th column(tsdall[:,3]): [x1-x0, x2-x1, ..., x_{t-4}-x_{t-3}, x_{t-3}-x_{t-4}] are X variables (corresponding to coeff *alpha_3*)

    tsdshort[response variable]:
    * [x4-x3, x5-x4, ..., x_{t-1}-x_{t-2}, x_t-x_{t-1}] are the Y variables

    Regression:
    * x4-x3 = lambda*x3 + alpha1*(x3-x2) + alpha2*(x2-x1) + alpha3*(x1-x0) + mu + epsilon
    * x5-x4 = lambda*x4 + alpha1*(x4-x3) + alpha2*(x3-x2) + alpha3*(x2-x1) + mu + epsilon
    * ...
    * x_t-x_{t-1} = lambda*x_{t-1} + alpha1*(x_{t-1}-x_{t-2}) + alpha2*(x_{t-2}-x_{t-3}) + alpha3*(x_{t-3}-x_{t-4}) + mu + epsilon
    """
    # Maximum lag which is included in test, default 12*(nobs/100)^{1/4} 
    # Ref: https://www.statsmodels.org/stable/generated/statsmodels.tsa.stattools.adfuller.html
    if maxlag is None:
        maxlag = int(np.ceil(12 * (len(ts) / 100) ** (1 / 4)))

    # make sure we are working with an array, convert if necessary
    ts = np.asarray(ts)

    # Calculate the discrete difference
    tsdiff = np.diff(ts)

    # Create a 2d array of lags, trim invalid observations on both sides
    tsdall = lagmat(tsdiff[:, None], maxlag, trim='both', original='in')

    # Get dimension of the array
    nobs = tsdall.shape[0]

    # [explanatory variable] replace first column ([:,0]) xdiff with level of x
    tsdall[:, 0] = ts[-nobs - 1:-1]

    # [response variable]
    tsdshort = tsdiff[-nobs:]

    # Calculate the linear regression using an ordinary least squares model
    results = OLS(tsdshort, add_trend(tsdall[:, :maxlag + 1], 'c')).fit()

    # adfstat = DF = lambda / SE(lambda)
    adfstat = results.tvalues[0]

    # Get approx p-value from a precomputed table (from stattools)
    pvalue = mackinnonp(adfstat, 'c', N=1)

    return adfstat, pvalue
```

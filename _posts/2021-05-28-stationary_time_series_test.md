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

For the verification of our implementation we can compare the output of our implementation with the result of the function adfuller, included in the Python module statsmodels from the SciPy ecosystem, when used with the same input:

```
from statsmodels.tsa.stattools import adfuller

# Create a mean-reverting time series
np.random.seed(2021)
random_noise = np.random.randn(9999) / 100.
lambda_ = 0.2
mu = 5
y0 = 1
ts = [y0]
for n in range(9999):
    y1 = y0 + mu -lambda_*y0 + random_noise[n]
    ts.append(y1)
    y0 = y1

# ADF tests
print("ADF implemented:")
print(adf(ts))

print("ADF in statsmodels:")
print(adfuller(ts, autolag=None)) # autolag set to None to use maxlag
```

The results show that our implementation is consistent with adfuller method in statsmodels:

```
ADF implemented:
(-16.74395546457208, 1.351035439732185e-29)
ADF in statsmodels:
(-16.74395546457208, 1.351035439732185e-29, 38, 9961, {'1%': -3.4310066595695945, '5%': -2.861830204343065, '10%': -2.5669244706354584})
```

# Hurst Exponent
It is also possible to *determine the stationary nature of the time series* by measuring its speed of diffusion, because a stationary time series should diffuse from its initial value more slowly than a geometric random walk.

We can model the speed by the variance:

$$
\begin{equation} 
\textbf{Var}(\tau) = \left< |z(t+\tau) - z(t) |^2 \right> = \frac{1}{T} \int^T_0 |z(t+\tau) - z(t) |^2 
\end{equation}
$$

where $z$ is the observed values, $\tau$ is an arbitrary time lag, and $<…>$ is an average over all $t$’s. For a geometric random walk, we know that

$$
\begin{equation} 
\left< |z(t+\tau) - z(t) |^2 \right> \sim \tau
\end{equation}
$$

This relationship turns into an equality with some proportionally constant for a large value of $\tau$, but it may deviate for a small $\tau$. So if the time series is mean reverting this won’t hold.

Now we have to introduce the Hurst exponent H:

$$
\begin{equation} 
\left< |z(t+\tau) - z(t) |^2 \right> \sim \tau^{2H}
\end{equation}
$$

For a time series with a geometric random walk behaviour, $H=0.5$, for a mean reverting series, $H<0.5$, and, finally, for a trending series $H>0.5$. $H$ also is an indicator for the degree of mean reversion or trendiness: as $H$ decreases towards 0, the series is more mean reverting and as it increases towards 1, it is more trending.

There are $\color{blue}{\textbf{various methods}}$  of implementing a Hurst Exponent, depending on the definition (calculation) of variance. We introduce two commonly used methods here. One is defining the variance as square of standard deviations; another one is defining variance as square of **rescaled range**.

## Hurst exponent (1)
```
def hurst_exponent_1(ts):
    """Returns the Hurst Exponent of the time series vector ts
    Ref: 
    1. https://www.quantstart.com/articles/Basics-of-Statistical-Mean-Reversion-Testing/
    2. https://towardsdatascience.com/introduction-to-the-hurst-exponent-with-code-in-python-4da0414ca52e
    """
    N = len(ts)
    min_lag = 2
    max_lag = 100
    lags = range(min_lag, max_lag+1)

    # Calculate the array of the variances of the lagged differences
    tau = [np.square(np.std(np.subtract(ts[lag:], ts[:-lag]))) for lag in lags]

    # Use a linear fit to estimate the Hurst Exponent (2*H)
    poly = np.polyfit(np.log(lags), np.log(tau), 1)

    # Return the Hurst exponent from the polyfit output
    return poly[0]/2.0
```

## Hurst exponent (2)
```
def hurst_exponent_2(ts):
    """Returns the Hurst Exponent of the time series vector ts
    Ref:
    1. https://en.wikipedia.org/wiki/Hurst_exponent
    """
    # Convert into np.array if the original ts is not a np.array
    ts = np.array(ts)
    
    N = len(ts)
    min_lag = 10
    max_lag = int(np.floor(N/2))
    lags = range(min_lag, max_lag+1)

    # Calculate different rescaled ranges
    ARS = []
    for lag in lags:
        # divide ts into different samples
        num = int(np.floor(N/lag))
        RS = []
        for n in range(num):
            X = ts[n*lag:(n+1)*lag] 
            # [Important]: take lagged differences
            X_diff = X[1:] - X[:-1] # X_diff = np.diff(X)
            Y = X_diff - X_diff.mean()
            Z = np.cumsum(Y)
            R = Z.max() - Z.min()
            S = Y.std(ddof=1)
            RS.append(R/S)
        
        ARS.append(sum(RS)/len(RS))   
    
    # Note we are estimating rescaled range (RS), instead of
    # RS^2, so the factor 2 disappear here.
    poly = np.polyfit(np.log(lags), np.log(ARS), 1)
    return poly[0]
```

# Half-Life of Mean Reversion
Calculating the half-life of a mean reversion time series is very interesting because it gives us the measure of how long it takes to mean revert.
This measure is a way to interpret the $\lambda$ coefficient in the equation we have already seen:


$$
\begin{equation} 
\Delta x_t = \lambda x_{t-1} + \mu + \color{red}{\beta t + \alpha_1 \Delta x_{t-1} + ... + \alpha_k \Delta x_{t-k}  }+ \epsilon_t
\end{equation}
$$

For this new interpretation we have to transform this discrete time series into a differential form while ignoring the drift $\color{red}{\beta t}$ and the lagged differences $\color{red}{\alpha_1 \Delta x_{t-1} , ... , \alpha_k \Delta x_{t-k}  }$ and we get the following **Ornstein-Uhlenbeck formula** for the mean reverting process:

$$
\begin{equation} 
d x_t = (\lambda x_{t-1} + \mu)dt + d\epsilon_t
\end{equation}
$$

This form allows us to get an analytic solution for the $\color{blue}{\textbf{expected value}}$ of $x(t)$ (Note: $\textbf{E}\left(x(t)\right)=\textbf{E}\left(x(t-1)\right)$ here):

$$
\textbf{E}\left(x(t)\right) = x_0 \cdot\exp(\lambda t) - \frac{\mu}{\lambda}\cdot\left(1-\exp(\lambda t)\right)
$$

If $\color{blue}{\lambda \text{ is negative}}$ for a **mean-reverting process**, the expected value of the value decays exponentially to the value $-\mu/\lambda$ with the half-life of decay being $log(2)/\lambda$.

Finally, to calculate the half-life mean reversion test we can use this simple implementation in Python:

```
def half_life(ts):  
    """ 
    Calculates the half life of a mean reversion
    """
    # make sure we are working with an array, convert if necessary
    ts = np.asarray(ts)
    
    # delta = p(t) - p(t-1)
    delta_ts = np.diff(ts)
    
    # calculate the vector of lagged values. lag = 1
    lag_ts = np.vstack([ts[:-1], np.ones(len(ts[:-1]))]).T
   
    # calculate the slope of the deltas vs the lagged values 
    # Ref: https://numpy.org/doc/stable/reference/generated/numpy.linalg.lstsq.html
    lambda_, const = np.linalg.lstsq(lag_ts, delta_ts, rcond=None)[0]

    # compute and return half life
    # negative sign to turn half life to a positive value
    return - np.log(2) / lambda_
```

All this theory and practical examples in Python will help us to understand and master the concept of cointegration, which will be thoroughly discussed in a following post. Stay tuned!

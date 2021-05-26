---
title: Sample Mean and Variance
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

# Population

Suppose $\vec{X} $ is our population with $N$ elements:

$$\vec{X} = (X_1, X_2, X_3, ..., X_N)$$

According to the definition of mean and variance, we can have:

(1). population mean: 

$$
\begin{equation} \label{eq-pop-mean}
\mu = \textbf{E}(\vec{X}) = \frac{1}{N} \sum_{i=1}^{N}X_i
\end{equation}
$$

(2). population variance (thus standard deviation $\sigma$ is the square root of variance): 

$$
\begin{equation} \label{eq-pop-var}
\sigma^2 = \textbf{Var}(\vec{X}) = \frac{1}{N} \sum_{i=1}^{N} \left( X_i-\mu \right)^2
\end{equation}
$$

From the definitions above, we have the following conclusion: the variance equals **mean of square minus square of mean** (see [variance](https://en.wikipedia.org/wiki/Variance)):


$$
\begin{equation*} 
\textbf{Var}(\vec{X})  = \textbf{E}(\vec{X}^2) - \left[\textbf{E}(\vec{X})  \right]^2
\end{equation*}
$$

**Proof**

$$
\begin{align*} 
\textbf{Var}(\vec{X})  &= \frac{1}{N} \sum_{i=1}^{N} \left( X_i-\mu \right)^2 \\
                                                &= \frac{1}{N} \sum_{i=1}^{N} \left( X_i^2-2\cdot\mu\cdot X_i+\mu^2 \right) \\
																								&= \frac{1}{N} \sum_{i=1}^{N} X_i^2-2\cdot\mu\cdot \frac{1}{N} \sum_{i=1}^{N}X_i+\frac{1}{N} \sum_{i=1}^{N}\mu^2  \\
																								&= \frac{1}{N} \sum_{i=1}^{N} X_i^2-2\cdot\mu\cdot \mu+\mu^2  \\
																								&= \frac{1}{N} \sum_{i=1}^{N} X_i^2-\mu^2  \\
																								&= \textbf{E}(\vec{X}^2)-\left[\textbf{E}(\vec{X})  \right]^2  \\
\end{align*}
$$

Q.E.D
# Sample
We can randomly take out some samples from our population. If we fix the sample size to be $n$ (where $n<=N$), then there will be $C_N^n = \frac{N!}{n!\cdot(N-n)!}$ distinct samples from our population $\vec{X}$. 

As a brief example, let's say $N=4$, and $n=3$, then for a population $\vec{X} $:

$$\vec{X} = (X_1, X_2, X_3, X_4)$$

we can take out *four* distinct samples ($C_4^3 = 4$) as below (the probability of each is equal to $1/4$):

$$(X_1, X_2, X_3), (X_1, X_2, X_4), (X_1, X_3, X_4), (X_2, X_3, X_4)$$

And for each sample, we are able to calculate the mean of the sample (just the same as population mean while treating the sample as a new population):

$$
\begin{align*} 
\bar{X}_{sm,1} &= \frac{X_1 + X_2 + X_3} {3} \\ 
\bar{X}_{sm,2} &= \frac{X_1 + X_2 + X_4} {3} \\
\bar{X}_{sm,3} &= \frac{X_1 + X_3 + X_4} {3} \\
\bar{X}_{sm,4} &= \frac{X_2 + X_3 + X_4} {3} 
\end{align*} 
$$

where $sm$ stands for *sample mean*. For this **distribution of sample mean** $$\vec{X}_{sm} =\left( \bar{X}_{sm,1} , \bar{X}_{sm,2} , \bar{X}_{sm,3} , \bar{X}_{sm,4} \right)$$,   we can calculate its mean and variance:

(3). mean of sample mean: 

$$
\begin{equation} \label{eq-samplemean-mean}
\mu_{sm} = \textbf{E}(\vec{X}_{sm} ) = \frac{1}{4} \sum_{i=1}^{4} \bar{X}_{sm,i} 
\end{equation}
$$


(4). variance of sample mean:  

$$
\begin{equation} \label{eq-samplemean-var}
\sigma_{sm}^2 = \textbf{Var}(\vec{X}_{sm} ) = \frac{1}{4} \sum_{i=1}^{4} \left( \bar{X}_{sm,i} -\mu_{sm} \right)^2
\end{equation}
$$

For the relations of the original population and the sample mean, we have the following two assertations: 

(5). <span style="color:red">*mean of sample mean = population mean* </span>

$$
\begin{equation} 
\mu_{sm} = \textbf{E}(\vec{X}_{sm} ) \color{blue}{=\textbf{E}(\vec{X})  = \mu}
\end{equation}
$$

(6). <span style="color:red">*variance of sample mean = population variance / sample size* </span>

$$
\begin{equation} 
\sigma_{sm}^2 = \textbf{Var}(\vec{X}_{sm} ) \color{blue}{= \frac{\textbf{Var}(\vec{X}) }{n}=\frac{\sigma^2}{n} }
\end{equation}
$$

**Proof**

$$
\begin{align*} 
\mu_{sm} &= \textbf{E}(\vec{X}_{sm} )   \\
                     &= \frac{1}{4} \sum_{i=1}^{4} \bar{X}_{sm,i}  \\
										 &= \frac{1}{4} \left[ \frac{X_1 + X_2 + X_3} {3}  + \frac{X_1 + X_2 + X_4} {3} +  \frac{X_1 + X_3 + X_4} {3}  + \frac{X_2 + X_3 + X_4} {3}  \right] \\ 
										 & = \frac{1}{4} \left( \frac{\color{green}{3}\cdot X_1 + \color{green}{3} \cdot X_2 + \color{green}{3}\cdot X_3 + \color{green}{3}\cdot X_4} {3}   \right)    [\color{green}{\text{each element appears } C_{N-1}^{n-1}=C_3^2=3 \text{ times in the samples}}]\\
										 & = \frac{1}{\color{green}{C_N^n}} \left( \sum_{i=1}^{N}\frac{\color{green}{C_{N-1}^{n-1}}\cdot X_i} {\color{green}{n}}   \right) [\color{green}{\text{generalize to population size=N and sample size=n}}]\\
										 & = \frac{C_{N-1}^{n-1}}{C_N^n\cdot n}\sum_{i=1}^{N}X_i \\
										 &= \frac{1}{N}\sum_{i=1}^{N}X_i  [\color{green}{\text{when N=4, it is }\frac{1}{4} \left(  X_1  + X_2 + X_3 + X_4\right)} ]\\
                     &=\color{blue}{ \textbf{E}(\vec{X})  = \mu}
\end{align*}
$$

$$
\begin{align*} 
\sigma_{sm}^2 &= \textbf{Var}(\vec{X}_{sm} )  \\
                     &= \frac{1}{4} \sum_{i=1}^{4} \left( \bar{X}_{sm,i} -\mu_{sm} \right)^2 \\
										 &= \frac{1}{4} \sum_{i=1}^{4} \left( \bar{X}_{sm,i} -\mu \right)^2  [\color{green}{\text{as we have demonstrated } \mu_{sm} =\mu}]\\
										 &= \frac{1}{4} \left[ \left( \frac{X_1 + X_2 + X_3} {3} -\mu\right)^2 + 
										                                            \left( \frac{X_1 + X_2 + X_4} {3} -\mu\right)^2 +
																																\left( \frac{X_1 + X_3 + X_4} {3} -\mu\right)^2 +
																																\left( \frac{X_2 + X_3 + X_4} {3} -\mu\right)^2  \right]  \\
										&= \frac{1}{4} \left[ \left( \frac{(X_1-\mu) +(X_2-\mu) + (X_3-\mu)} {3} \right)^2 + 
										                                            \left( \frac{(X_1-\mu) +(X_2-\mu) + (X_4-\mu)} {3} \right)^2 +
																																\left( \frac{(X_1-\mu) +(X_3-\mu) + (X_4-\mu)} {3} \right)^2 +
																																\left( \frac{(X_2-\mu) +(X_3-\mu) + (X_4-\mu)} {3} \right)^2  \right]  \\
										&= \frac{1}{4} \left[ \left( \frac{\color{green}{3}\cdot(X_1-\mu)^2 +
																																												\color{green}{3}\cdot(X_2-\mu)^2 + 
																																												\color{green}{3}\cdot(X_3-\mu)^2 + 
																																												\color{green}{3}\cdot(X_4-\mu)^2 \color{red}{+0}} {3^ 2} \right)  \right] 
																															 [\color{green}{\text{each element appears } C_{N-1}^{n-1}=C_3^2=3 \text{ times in the samples}}, 
																															 \color{red}{\text{assuming all $X_i$ are uncorrelated, so cross terms are neligible}}]\\
										& = \frac{1}{\color{green}{C_N^n}} \left( \sum_{i=1}^{N}\frac{\color{green}{C_{N-1}^{n-1}}\cdot(X_i-\mu)^2} {\color{green}{n^2}}   \right)
										                                          [\color{green}{\text{generalize to population size=N and sample size=n}}]\\
										& = \frac{C_{N-1}^{n-1}}{C_N^n\cdot n^2}\sum_{i=1}^{N}(X_i-\mu)^2 \\
										& = \frac{1}{n}\cdot\frac{1}{N}\sum_{i=1}^{N}(X_i-\mu)^2 \\
										&=\frac{1}{n}\cdot \textbf{Var}(\vec{X})  \\
										&=\color{blue}{\frac{\textbf{Var}(\vec{X}) }{n}=\frac{\sigma^2}{n} }
\end{align*}
$$

Q.E.D

# Variance of a sum equals the sum of the variances?

**Note**: in the proof above, we have used the conclusion that **the variance of  a sum equals the sum of the variances**:

$$
\begin{equation*}
\textbf{Var} \left(  \sum_{i=1}^{N}X_i \right)=  \sum_{i=1}^{N} \textbf{Var} (X_i) \color{red}{?}
\end{equation*}
$$

This conclusion is *NOT true*  in general. There is a [post](https://stats.stackexchange.com/questions/31177/does-the-variance-of-a-sum-equal-the-sum-of-the-variances) that explains this issue. To summarise, starting from the calculation of variance as discussed above: $\textbf{Var}(\vec{X})  = \textbf{E}(\vec{X}^2) - \left[\textbf{E}(\vec{X})  \right]^2$, we can show that

$$
\begin{align*}
\textbf{Var} \left(  \sum_{i=1}^{N}X_i \right) &= \textbf{E} \left(  \left[ \sum_{i=1}^{N}X_i \right]^2\right) - \left[ \textbf{E} \left(  \sum_{i=1}^{N}X_i \right)  \right]^2 \\
                                                                                                  &=\textbf{E} \left(   \sum_{i=1}^{N}\sum_{j=1}^{N}X_i\cdot X_j\right) - \sum_{i=1}^{N}\sum_{j=1}^{N}\textbf{E} (X_i)\cdot \textbf{E} (X_j) \\
																																																	&=\sum_{i=1}^{N}\sum_{j=1}^{N}\textbf{E} \left(   X_i\cdot X_j\right) - \sum_{i=1}^{N}\sum_{j=1}^{N}\textbf{E} (X_i)\cdot \textbf{E} (X_j) \\
																																																	&=\sum_{i=1}^{N}\sum_{j=1}^{N}\left(\textbf{E} \left(   X_i\cdot X_j\right) - \textbf{E} (X_i)\cdot \textbf{E} (X_j) \right)\\
																																																	&=\sum_{i=1}^{N}\sum_{j=1}^{N}\textbf{Cov}(X_i, X_j)\\
\end{align*}
$$

Therefore,

**if the variables are uncorrelated**, that is, $\textbf{Cov}(X_i, X_j)=0$ for $i\neq j$, then

$$
\begin{equation*}
\textbf{Var} \left(  \sum_{i=1}^{N}X_i \right)  =\sum_{i=1}^{N}\sum_{j=1}^{N}\textbf{Cov}(X_i, X_j) = \sum_{i=1}^{N}\textbf{Cov}(X_i, X_i) = \sum_{i=1}^{N}\textbf{Var}(X_i)
\end{equation*}
$$

**but if the variables are correlated**, this relation fails in general: for example, suppose $X_1$, $X_2$ are two random variables each with variance $\sigma^2$ and $\textbf{Cov}(X_1,X_2)=\rho$ where $0<\rho<\sigma^2$. Then $\textbf{Var}(X_1 + X_2) = 2\cdot(\sigma^2 + \rho) \neq 2\cdot \sigma^2$, so the identity fails.
# Unbiased estimator for population variance
When computing variance from a sample, we use

$$
\begin{equation*} 
s^2 = \frac{1}{\color{blue}{n-1}} \sum_{i=1}^{n} \left( X_{sm},i-\mu_{sm} \right)^2
\end{equation*}
$$

as an*unbiased estimator for the population variance* , which means, $\textbf{E}(s^2)=\sigma^2$. Why do we use $n-1$ instead of $n$ here? It is an issue related to [Bessel's correction](https://en.wikipedia.org/wiki/Bessel%27s_correction). 

To explain this, let's take $\textbf{E}(\vec{X}^2) =\textbf{Var}(\vec{X})  + \left[\textbf{E}(\vec{X})  \right]^2$ as assumed knowledge, and assume that variance of a sum equals the sum of the variances (i.e., variables are uncorrelated).

$$
\begin{align*} 
\textbf{E}\left( \sum_{i=1}^{n} \left( X_{sm,i}-\mu_{sm} \right)^2 \right) 
									&= \textbf{E}\left( \sum_{i=1}^{n}X_{sm,i}^2 - 2\cdot \mu_{sm}\cdot\sum_{i=1}^{n} X_{sm,i} + n\cdot\mu_{sm}^2 \right) \\
									&= \textbf{E}\left( \sum_{i=1}^{n}X_{sm,i}^2 - n\cdot\mu_{sm}^2 \right)  
									[\color{green}{\textbf{E}(X_{sm,i} )=\mu_{sm}, \text{ and } \textbf{E}(\mu_{sm}\cdot X)=\mu_{sm}\cdot\textbf{E}(X)}]\\
									&=n\cdot\textbf{E}(X_{sm,i}^2) - n\cdot\textbf{E}(\mu_{sm}^2) \\
									&=n\cdot\left[ \textbf{Var}(X_{sm,i}) + \left(\textbf{E}(X_{sm,i})\right)^2 \right]- n\cdot\left[ \textbf{Var}(\mu_{sm}) + \left(\textbf{E}(\mu_{sm})\right)^2 \right]
									[\color{green}{\textbf{E}(\vec{X}^2) =\textbf{Var}(\vec{X})  + \left[\textbf{E}(\vec{X})  \right]^2}] \\
									&=n\cdot\left(\sigma^2 + \mu_{sm}^2\right) - n\cdot\left(\frac{\sigma^2}{n} + \mu_{sm}^2\right) 
									[\color{green}{\textbf{E}(X_{sm,i})=\mu_{sm}=\textbf{E}(\mu_{sm}); \text{ and } \textbf{Var}(\mu_{sm})=\textbf{Var}(X_{sm,i})/n=\sigma^2/n} \color{red}{ \text{ if we assume variables $X$ are uncorrelated as disscussed above}} ] \\
									&=(n-1)\cdot\sigma^2\\
\end{align*}
$$

Therefore we have:

$$
\begin{equation*} 
\textbf{E}\left( \frac{1}{\color{blue}{n-1}}\sum_{i=1}^{n} \left( X_{sm,i}-\mu_{sm} \right)^2 \right) = \sigma^2
\end{equation*}
$$

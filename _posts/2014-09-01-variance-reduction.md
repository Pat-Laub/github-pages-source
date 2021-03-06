---
layout: post
title: Variance reduction techniques
subtitle: Antithetic sampling, control variates, and stratification
tags: [math, monte carlo]
author: pjl
excerpt: >
    Notes made for an informal lecture on the Monte Carlo tricks available
    for variance reduction. This was made for a reading course when we
    worked through Søren's Monte Carlo book.
mathjax: true
---


Antithetic Sampling
===================

A typical CMC estimator for $l = {\mathbb{E}}[Z]$ is to take $N$ iid
replications $\\{Z_1, Z_2, \dots, Z_N\\}$ and use their sample average,
i.e.

$$ \hat{l}_{CMC} = \frac{1}{N} \sum_{i=1}^N Z_i. $$

This estimator will have variance

$$
\begin{align}
\sigma^2_{CMC} &= \frac{1}{N^2} \mathrm{Var}\left[\sum_{i=1}^N Z_i\right] \\
&= \frac{N}{N^2} \mathrm{Var}[Z_1] \\
\Rightarrow \sigma^2_{CMC} &= \frac{\mathrm{Var}[Z_1]}{N}
\end{align}
$$


Another approach is to not sample $N$ random variables independently,
but $N/2$ pairs of variables which have negative correlation. I.e.
sample

$$\{ (Z_1, Z^*_1), (Z_2, Z^*_2), \dots, (Z_{N/2}, Z^*_{N/2}) \}$$

where they are all of the same distribution and independent except for
$Z_i$ with $Z^*_i$. The estimator is then

$$\begin{align}
    \hat{l}_{AS} &= \frac{1}{N/2} \sum_{i=1}^{N/2} \frac{Z_i+Z^*_i}{2} \\
        &= \frac{1}{N} \sum_{i=1}^{N/2} Z_i+Z^*_i.\end{align}$$

This has variance

$$\begin{align}
\sigma^2_{AS} &= \frac{1}{N^2} {\mathrm{Var}}\left[\sum_{i=1}^{N/2} Z_i+Z^*_i \right] \\
    &= \frac{N/2}{N^2} {\mathrm{Var}}\left[Z_1+Z^*_1] \right]  \\
    &= \frac{1}{2N} \left[{\mathrm{Var}}[Z_1]+{\mathrm{Var}}[Z^*_1] + 2{\mathrm{Cov}}[Z_1, Z^*_1] \right]  \\
    &= \frac{1}{2N} \left[2{\mathrm{Var}}[Z_1] + 2{\mathrm{Cov}}[Z_1, Z^*_1] \right]  \\
    &= \frac{\mathrm{Var}[Z_1] + {\mathrm{Cov}}[Z_1, Z^*_1]}{N} \\
\Rightarrow \sigma^2_{AS} &= \sigma^2_{CMC} + \frac{\mathrm{Cov}[Z_1, Z^*_1]}{N}\end{align}$$

Therefore

$${\mathrm{Cov}}[Z_i, Z^*_i] < 0 \Rightarrow \sigma^2_{AS} < \sigma^2_{CMC}.$$

Say that random variables $X$ and $Y$ have the same marginal
distributions ($F_X(z) = F_Y(z) = F(z)$) but are generated using
$U_i {\overset{iid}{\sim}}U(0,1)$ by $X_i = F^{-1}(U_i)$ and
$Y_i = F^{-1}(1-U_i)$ then ${\mathrm{Cov}}(X,Y)$ is minimal (see Theorem
5.3 in Ripley 2009, p. 130).\
Introduce dependence (specifically negative correlation) in your
iterates to reduce variance.\

Control Variates
================

Say a simulation produces an output $Y$, and the same simulation also
outputs another random variable $\tilde{Y}$ which is correlated with $Y$
and whose expectation $\tilde{l} = {\mathbb{E}}[\tilde{Y}]$ is known.
The $\tilde{Y}$ is called a control variable for $Y$. Use the estimator

$$\hat{l}_{CV}  = \frac{1}{N} \sum_{k=1}^N \left[ Y_k - \alpha (\tilde{Y_k} - \tilde{l}) \right].$$

This will have variance

$$\begin{align}
 \sigma^2_{CV} &= \frac{1}{N^2} {\mathrm{Var}}\left[ \sum_{k=1}^N \left[ Y_k - \alpha (\tilde{Y_k} - \tilde{l}) \right] \right] \\
 &= \frac{1}{N^2} \left[ \sum_{k=1}^N {\mathrm{Var}}\left[ Y_k - \alpha (\tilde{Y_k} - \tilde{l}) \right] \right] \\
  &= \frac{N}{N^2} \left[ {\mathrm{Var}}\left[ Y_1 - \alpha (\tilde{Y_1} - \tilde{l}) \right] \right] \\
  &= \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] + {\mathrm{Var}}[-\alpha (\tilde{Y_1} - \tilde{l})] + 2{\mathrm{Cov}}[Y_1, -\alpha (\tilde{Y_1} - \tilde{l})] \right] \\
  &= \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] + (-\alpha)^2 {\mathrm{Var}}[\tilde{Y_1}] + 2{\mathrm{Cov}}[Y_1, -\alpha (\tilde{Y_1} - \tilde{l})] \right] \\
 \end{align}$$

Note that:

$$\begin{align}
    {\mathrm{Cov}}[Y_1, -\alpha(\tilde{Y}_1 - \tilde{l})] &= {\mathbb{E}}[(Y_1-{\mathbb{E}}[Y_1])(-\alpha(\tilde{Y}_1 - \tilde{l}) - {\mathbb{E}}[-\alpha(\tilde{Y}_1 - \tilde{l})])] \\
    &= {\mathbb{E}}[(Y_1-{\mathbb{E}}[Y_1])(-\alpha)(\tilde{Y}_1 - \tilde{l})] \\
    &= -\alpha {\mathbb{E}}[(Y_1-{\mathbb{E}}[Y_1])(\tilde{Y}_1 - \tilde{l})] \\
    &= -\alpha {\mathbb{E}}[(Y_1-{\mathbb{E}}[Y_1])(\tilde{Y}_1 - {\mathbb{E}}[\tilde{Y}_1])] \\
    &= -\alpha {\mathrm{Cov}}[Y_1, \tilde{Y}_1] \\\end{align}$$

So therefore

$$\sigma^2_{CV} = \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] + \alpha^2 {\mathrm{Var}}[\tilde{Y_1}] - 2\alpha {\mathrm{Cov}}[Y_1, \tilde{Y_1}] \right]$$

To find the $\alpha$ which minimises the variance then look for
stationary points

$$\begin{align}
 \frac{\mathrm{d} \sigma^2_{CV}}{\mathrm{d} \alpha} = 0 &= \frac{1}{N} \left[ 2\alpha {\mathrm{Var}}[\tilde{Y_1}] - 2{\mathrm{Cov}}[Y_1, \tilde{Y_1}] \right] \\
 \Rightarrow 0 &= \left[ \alpha {\mathrm{Var}}[\tilde{Y_1}] - {\mathrm{Cov}}[Y_1, \tilde{Y_1}] \right] \\
 \Rightarrow \alpha {\mathrm{Var}}[\tilde{Y_1}] &= {\mathrm{Cov}}[Y_1, \tilde{Y_1}] \\
 \Rightarrow \alpha  &= \frac{\mathrm{Cov}[Y_1, \tilde{Y_1}]}{\mathrm{Var}[\tilde{Y_1}]} \\
 \end{align}$$

So the minimum variance achieved by using control variates is

$$\begin{align}
    \hat{\sigma}^2_{CV} &= \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] + \left(\frac{\mathrm{Cov}[Y_1, \tilde{Y_1}]}{\mathrm{Var}[\tilde{Y_1}]}\right)^2 {\mathrm{Var}}[\tilde{Y_1}] - 2\left(\frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]}{ {\mathrm{Var}}[\tilde{Y_1}]}\right) {\mathrm{Cov}}[Y_1, \tilde{Y_1}] \right] \\
    &= \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] + \left(\frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]^2}{ {\mathrm{Var}}[\tilde{Y_1}]}\right) - 2\left(\frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]^2}{ {\mathrm{Var}}[\tilde{Y_1}]}\right) \right] \\
    &= \frac{1}{N} \left[ {\mathrm{Var}}[Y_1] - \left(\frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]^2}{ {\mathrm{Var}}[\tilde{Y_1}]}\right) \right] \\
    &= \frac{ {\mathrm{Var}}[Y_1]}{N} - \frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]^2}{N{\mathrm{Var}}[\tilde{Y_1}]} \\\end{align}$$

$$\Rightarrow \hat{\sigma}^2_{CV} = \sigma^2_{CMC} - \frac{ {\mathrm{Cov}}[Y_1, \tilde{Y_1}]^2}{N{\mathrm{Var}}[\tilde{Y_1}]}$$

$$\Rightarrow \hat{\sigma}^2_{CV} < \sigma^2_{CMC}$$

For each simulation find another variable – the control variable – that
is highly correlated (positively or negatively) with the variable of
interest which has known expectation (and preferably small variance).

Stratification
==============

Stratified sampling uses the form $l = {\mathbb{E}}[{\mathbb{E}}[Y|Z]]$
and calculates the ${\mathbb{E}}[Y|Z]$ by partitioning the state space
$\Omega$ into $\{Z=i\}, i=1, \dots, m$, the sections called *strata*.
The general equation utilised is the law of total expectation, i.e.

$${\mathbb{E}}[Y|Z] = \sum_{i=1}^m {\mathbb{E}}[Y|Z=i] {\mathbb{P}}(Z=i).$$

Say that ${\mathbb{P}}(Z=i)=p_i$ are known constants, then for each
strata choose $N_i$ (where $\sum_i N_i = N$) as the number to sample
from that strata, and use the estimator:

$$l_{S} = \sum_{i=1}^m p_i \frac{1}{N_i} \sum_{j=1}^{N_i} Y_{j|Z=i}$$

Note that we are sampling from the conditional distributions of $Y$
given $Z=i$. The variance of this estimator is

$$\begin{align}
    \sigma^2_{S} &= {\mathrm{Var}}\left[\sum_{i=1}^m p_i \frac{1}{N_i} \sum_{j=1}^{N_i} Y_{j|Z=i}\right] \\
    &= \sum_{i=1}^m \frac{p_i^2}{N_i^2} {\mathrm{Var}}\left[\sum_{j=1}^{N_i} Y_{j|Z=i}\right]\end{align}$$

So say ${\mathrm{Var}}[Y_{j \mid Z=i}] = \sigma^2_i$ then

$$\sigma^2_{S} = \sum_{i=1}^m \frac{p_i^2\sigma^2_i}{N_i^2}$$

In order to minimise the variance then select the $N_i$ by the rule

$$N_i = N \frac{p_i \sigma_i}{\sum_{k=1}^m p_k \sigma_k} , \qquad i=1,\dots,m.$$

However it is rare that the variance of each strata is known so a pilot
run can be used to estimate $\sigma_i$ using the sample variance.
If using $N_i$ selected optimally as above, then the best variance of
the estimator is then

$$\hat{\sigma}^2_{S} = \frac{1}{N} \left( \sum_{i=1}^m p_i \sigma_i \right)^2$$

Look for different “classes” in your state space (i.e strata) that
partition the space, and have small variance, so that you can sample
more effectively the noisy parts of the state space and infrequently
sample the less noisy parts.
See Ripley (2009, Chapter 5) for a detailed look at stratified sampling
in statistics/survey theory.

References:
-   Cochran, W. G. (2007). Sampling techniques. John Wiley & Sons.
-   Ripley, B. D. (2009). Stochastic simulation (Vol. 316). John Wiley &
    Sons.
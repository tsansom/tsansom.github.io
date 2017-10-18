---
layout: post
comments: true
title: "Deriving the Simple Linear Regression Coefficients"
data: 2017-10-18 07:31:34 -0500
categories: statistical machine learning simple linear regression coefficients derivation ISLR
---

In this post I will walk through the derivation of the coefficients for
simple linear regression as part of my ongoing work through of
[*An Introduction to Statistical Learning with Applications in
R*](http://www-bcf.usc.edu/~gareth/ISL/). As in the [Bias-Variance
Tradeoff](https://tsansom.github.io/statistical/learning/mse/bias/variance/derive/2017/10/17/deriving-bias-variance-tradeoff.html) derivation, I'll be
using notation in the flavor of ISLR for convenience and consistency in my notes.  

## Simple Linear Regression
This is a very straightforward approach for predicting a quantitative
response $Y$ on the basis of a single predictor $X$. It assumes that there
is approximately a linear relationship between the predictor, $X$, and
the response, $Y$:  

$$Y \approx \beta_0 + \beta_1X$$  

$\beta_0$ and $\beta_1$ are the unknown coefficients that represent
*intercept* and *slope* respectively in the linear model. We can use
our training data to produce estimates for $\hat{\beta}_0$ and
$\hat{\beta}_1$ and then make predictions with:  

$$\hat{y} = \hat{\beta}_0 + \hat{\beta}_1x$$  

where $\hat{y}$ indicates a prediction of $Y$ on the basis of $X = x$.
The $\hat{ }$ symbol denotes the estimated value for an unknown parameter,
coefficient, or predicted value of a response.  

## Estimating the Coefficients
Since $\beta_0$ and $\beta_1$ are usually unknown, we must use data to
estimate them. Let $(x_1, y_1), (x_2, y_2), ..., (x_n, y_n)$ be the $n$
observation pairs. Our goal is to obtain coefficient estimates
$\hat{\beta}_0$ and $\hat{\beta}_1$ such that the linear model fits
the available data well. That is, $y_i \approx \hat{\beta}_0 +
\hat{\beta}_1x_i$ for $i = 1, 2, ..., n$. The most common approach for
this involves minimizing the *least squares* criterion.  

Let $\hat{y}_i = \hat{\beta}_0 + \hat{\beta}_1x_i$ be the
prediction for $Y$ bassed on the $i$th value of $X$. Then $\epsilon = y_i - \hat{y}_i$ represents the *residual error*, or the difference
 between the $i$th observed response and the $i$th predicted response
 from the linear model. We then define the *residual sum of squares* (RSS)
 as:  

 $$RSS = \epsilon_1^2 + \epsilon_2^2 + ... + \epsilon_n^2$$  

or equivalently:  

$$RSS = (y_1 - \hat{\beta}_0 - \hat{\beta}_1x_1)^2 +
(y_2 - \hat{\beta}_0 - \hat{\beta}_1x_1)^2 + ... +
(y_n - \hat{\beta}_0 - \hat{\beta}_1x_n)^2$$  

which is the same as:  

$$\sum_{i=1}^n(y_i - \hat{\beta}_0 - \hat{\beta}_1x_i)^2$$  

As mentioned earlier, the *least squares* criterion aims to minimize the
RSS. The RSS will be minimized at the values of $\hat{\beta}_0$ and
$\hat{\beta}_1$ for which
$\frac{\partial{RSS}}{\partial{\hat{\beta}_0}} = 0$ and
$\frac{\partial{RSS}}{\partial{\hat{\beta}_1}} = 0$. Let's start with
$\frac{\partial{RSS}}{\partial{\hat{\beta}_0}}$:  

$$\frac{\partial{RSS}}{\partial{\hat{\beta}_0}} =
\sum_{i=1}^n2(y_i - \hat{\beta}_0 - \hat{\beta}_1x_i)
 = 2(n\hat{\beta}_0 + \hat{\beta}_1\sum_{i=1}^nx_i -
\sum_{i=1}^ny_i) = 0$$  

If we divide through by 2 and $n$ then use the sample means definitions
$\bar{x} \equiv \frac{1}{n}\sum_{i=1}^nx_i$ and
$\bar{y} \equiv \frac{1}{n}\sum_{i=1}^ny_i$, the equation can be rewritten
as:  

$$\hat{\beta}_0 = \bar{y} - \hat{\beta}_1\bar{x}$$  

Now for $\frac{\partial{RSS}}{\partial{\hat{\beta}_1}}$:  

$$\frac{\partial{RSS}}{\partial{\hat{\beta}_1}} =
\sum_{i=1}^n-2x_i(y_i - \hat{\beta}_0 - \hat{\beta}_1x_i) =
\sum_{i=1}^n-2(x_iy_i - \hat{\beta}_0x_i - \hat{\beta}_1x_i^2) =
0$$  

Substituting the equation for $\hat{\beta}_0$ from above into this
equation we get:  

$$\sum_{i=1}^n(x_iy_i - x_i\bar{y} - \hat{\beta}_1x_i\bar{x} -
\hat{\beta}_1x_i^2) = 0$$  

This can be separated into two sums:  

$$\sum_{i=1}^n(x_iy_i - x_i\bar{y}) -
\hat{\beta}_1\sum_{i=1}^n(x_i^2 - x_i\bar{x}) = 0$$

Solving for $\hat{\beta}_1$ we get:  

$$\hat{\beta}_1 =
\frac{\sum_{i=1}^nx_iy_i - x_i\bar{y})}{\sum_{i=1}^n(x_i^2 - x_i\bar{x})}
 = \frac{\sum_{i=1}^n(x_iy_i) - n\bar{x}\bar{y}}{\sum_{i=1}^n(x_i^2) -
n\bar{x}^2}$$  

Using these zero identities:  

$$\sum_{i=1}^n(\bar{x}^2 - x_i\bar{x}) = 0 \qquad\text{and}\qquad
\sum_{i=1}^n(\bar{x}\bar{y} - y_i\bar{x}) = 0$$  

we can translate the previous equation into a more intitively obvious
form:  

$$\hat{\beta}_1 = \frac{\sum_{i=1}^n(x_iy_i - x_i\bar{y}) +
\sum_{i=1}^n(\bar{x}\bar{y} - y_i\bar{x})}
{\sum_{i=1}^n(x_i^2 - x_i\bar{x}) +
\sum_{i=1}^n(\bar{x}^2 - x_i\bar{x})}$$  

This equation can be further factorized into the final form for
$\hat{\beta}_1$:  

$$\boxed{\hat{\beta}_1 = \frac{\sum_{i=1}^n(x_i - \bar{x})
(y_i - \bar{y})}{\sum_{i=1}^n(x_i - \bar{x})^2}}$$  

And $\hat{\beta}_0$ from earlier:  

$$\boxed{\hat{\beta}_0 = \bar{y} - \hat{\beta}_1\bar{x}}$$  

These two equations define the *least squares coefficient estimates* for
simple linear regression.

That's it, let me know if you have any questions or problems by leaving
comments below.

Cheers!

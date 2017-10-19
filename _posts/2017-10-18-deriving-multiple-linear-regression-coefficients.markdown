---
layout: post
comments: true
title: "Deriving the Multiple Linear Regression Coefficients"
data: 2017-10-18 08:58:05 -0500
categories: statistical machine learning multiple linear regression coefficients derivation ISLR
---

In this post I will walk through the derivation of the coefficients for
multiple linear regression as part of my ongoing work through of
[*An Introduction to Statistical Learning with Applications in
R*](http://www-bcf.usc.edu/~gareth/ISL/). As in the [Bias-Variance
Tradeoff](https://tsansom.github.io/statistical/learning/mse/bias/variance/derive/2017/10/17/deriving-bias-variance-tradeoff.html) derivation, I'll be
using notation in the flavor of ISLR for convenience and consistency in my
notes.  

## Multiple Linear Regression
In simple linear regression we are predicting a quantitative response $Y$
on the basis of a single predictor $X$. This relationship takes the form:  

$$Y = \beta_0 + \beta_1X + \epsilon$$  

Normally, we have more than one predictor to model the responses as opposed
to a single predictor for simple linear regression. The approach of fitting
a separate simple linear regression model for each predictor is not
satisfactory since (a) it is unclear how to make a single prediction of sales
given multiple predictors and (b) each of the regression equations ignores
the other predictors when forming estimates for the regression coefficients.
If the predictors are in any way correlated with each other, this can lead to
very misleading estimates of the response.  

Instead, we will extend the simple linear regression model so that it can
directly accommodate multiple predictors. We do this by giving each predictor
a separate slope coefficient in a single model. If we have $p$ distinct
predictors, the multiple linear regression model takes the form:  

$$Y = \beta_0 + \beta_1X_1 + \beta_2X_2 + ... + \beta_pX_p + \epsilon,$$  

where $X_j$ represents the $j$th predictor and $\beta_j$ quantifies the
association between that variable and the response. We interpret $\beta_j$
as the average effect on $Y$ of a one unit increase in $X_j$ holding all
other predictors fixed.  

Since the actual regression coefficients $\beta_0, \beta_1, ..., \beta_p$
are unknown, they must be estimated with $\widehat{\beta}_0,
\widehat{\beta}_1, ..., \widehat{\beta}_p$. Then we can make predictions
using the formula:  

$$\widehat{y} = \widehat{\beta}_0 + \widehat{\beta}_1x_1 +
\widehat{\beta}_2x_2 + ... + \widehat{\beta}_px_p$$  

The parameters are estimated using the same *least squares* approach
as in simple linear regression. We choose $\widehat{\beta}_0,
\widehat{\beta}_1, ..., \widehat{\beta}_p$ to minimize the sum of squared
residuals:  

$$RSS = \sum_{i=1}^n(y_i - \widehat{y}_i)^2$$  

$$= \sum_{i=1}^n(y_i - \widehat{\beta}_0 - \widehat{\beta}_1x_{i1} -
\widehat{\beta}_2x_{i2} - ... - \widehat{\beta}_px_{ip})^2$$  

or equivalently  

$$= \sum_{i=1}^n\biggr(y_i - (\widehat{\beta}_0 +
\sum_{j=1}^p\widehat{\beta}_jX_{ij})\biggr)^2$$  

## Deriving the Multiple Linear Regression Coefficients  
Because the multiple regression coefficient estimates take on a somewhat
complicated form, they are best represented using matrix algebra. Let  

$$
\textbf{y} = \begin{bmatrix}
y_1\\
y_2\\
\vdots
\\y_n
\end{bmatrix}
\qquad\text{,}\qquad
\textbf{X} = \begin{bmatrix}
1&x_{11}&x_{12}&\cdots&x_{1p}\\
1&x_{21}&x_{22}&\cdots&x_{2p}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&x_{n1}&x_{n2}&\cdots&x_{np}
\end{bmatrix}
$$  

$$
\widehat{\beta} = \begin{bmatrix}
\widehat{\beta_0}\\
\widehat{\beta_1}\\
\vdots\\
\widehat{\beta_p}
\end{bmatrix}
\qquad\text{,}\qquad
\epsilon = \begin{bmatrix}
\epsilon_1\\
\epsilon_2\\
\vdots\\
\epsilon_n
\end{bmatrix}
$$  

With this compact notation, we can rewrite the multiple linear regression
formula as:  

$$\textbf{y} = \textbf{X}\beta + \epsilon$$  

and the RSS as:  

$$RSS = \sum_{i=1}^n\epsilon^2 = \sum_{i=1}^n(\textbf{y} -
\widehat{\beta}\textbf{X})^2$$  

which in linear algebra is equivalent to:  

$$RSS = \epsilon^t\epsilon = (\textbf{y} -
\widehat{\beta}\textbf{X})^t(\textbf{y} - \widehat{\beta}\textbf{X})$$  

$$= (\textbf{y}^t - \widehat{\beta}^t\textbf{X}^t)(\textbf{y} -
\widehat{\beta}\textbf{X})$$  

$$= \textbf{y}^t\textbf{y} - \widehat{\beta}^t\textbf{X}^ty -
\textbf{y}^t\textbf{X}\widehat{\beta} +
\widehat{\beta}^t\textbf{X}^t\textbf{X}\widehat{\beta}$$  

and because the transpose of a $1 \times 1$ matrix is the same term,
e.g. $\widehat{\beta}^t\textbf{X}^t\textbf{y} =
(\widehat{\beta}^t\textbf{X}^t\textbf{y})^t =
\textbf{y}^t\textbf{X}\widehat{\beta}$ we are left with:  

$$RSS = \textbf{y}^t\textbf{y} -
2\widehat{\beta}^t\textbf{X}^t\textbf{y} +
\widehat{\beta}^t\textbf{X}^t\textbf{X}\widehat{\beta}$$  

As with simple linear regression we want to choose the coefficients such
that $\frac{\partial{RSS}}{\partial\widehat{\beta}} = 0$. *Note: there is
only one equation here unlike the simple linear regression case since
$\widehat{\beta}_0$ is the first value of the $\widehat{\beta}$ matrix.*
Mathematically, this becomes:  

$$\frac{\partial{RSS}}{\partial\widehat{\beta}} =
-2\textbf{X}^t\textbf{y} + 2\textbf{X}^t\textbf{X}\widehat{\beta} = 0$$  

Dividing by 2 and rearranging, we get:  

$$\textbf{X}^t\textbf{X}\widehat{\beta} = \textbf{X}^t\textbf{y}$$  

Now, multiplying both sides by the inverse of $\textbf{X}^t\textbf{X}$
we get the final form of $\widehat{\beta}$ as:  

$$\boxed{\widehat{\beta} =
(\textbf{X}^t\textbf{X})^{-1}\textbf{X}^t\textbf{y}}$$  

That's it, let me know if you have any questions or problems by leaving
comments below.

Cheers!

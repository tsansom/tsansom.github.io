---
layout: post
comments: true
title: "Deriving the Bias-Variance Tradeoff"
date: 2017-10-17 05:11:59 -0500
categories: statistical learning mse bias variance derive
---

At the moment, I'm doing a complete work through of [*An Introduction to
  Statistical Learning with Applications in R*](http://www-bcf.usc.edu/~gareth/ISL/)
  by James, Witten, Hastie, and Tibshirani except that I'm recreating all the
  figures and excercises in python. In some places its stated that proving an
  equation is beyond the scope of the book. In an effort to fully understand
  the underlying statistics and mathematics behind statistical learning, I am
  digging a bit deeper and doing the derivations myself.

In this post, I'll be walking through the derivation of the *bias-variance tradeoff*
  which comes from the *mean square error* equation. I'll be using notation
  in the flavor of ISLR for convenience and consistency in my notes. Let's start
  with MSE:  

$$\mathbb{E}[(y_0 - \widehat{y}_0)^2] = \mathbb{E}[y_0^2 - 2y_0\widehat{y}_0 + \widehat{y}_0^2]$$  

$$= \mathbb{E}[y_0]^2 - 2\mathbb{E}[y_0\widehat{y}_0] + \mathbb{E}[\widehat{y}_0^2]$$  

Apply this lemma \\( \mathbb{E}[x] = \mathbb{E}[(x - \mathbb{E}[x])^2] + \mathbb{E}[x]^2 \\)
  to the first and third terms to get:  

$$= \mathbb{E}[(y_0 - \mathbb{E}[y_0])^2] + \mathbb{E}[y_0]^2 - 2\mathbb{E}[y_0\widehat{y}_0]
+ \mathbb{E}[(\widehat{y}_0 - \mathbb{E}[\widehat{y}_0])^2] + \mathbb{E}[\widehat{y}_0]^2$$  

Use the fact that \\( \mathbb{E}[\widehat{y}_0] = \widehat{f}(x_0) \\) to obtain:  

$$= \mathbb{E}[(y_0 - \mathbb{E}[y_0])^2] + \mathbb{E}[y_0]^2 - 2\mathbb{E}[y_0]\widehat{f}(x_0)
+ \mathbb{E}[(\widehat{y}_0 - \widehat{f}(x_0))^2] + \widehat{f}(x_0)^2$$  

Rearranging this equation we get:  

$$= \mathbb{E}(y_0 - \mathbb{E}[y_0])^2] + \mathbb{E}[y_0]^2 - 2\mathbb{E}[y_0]\widehat{f}(x_0)
+ \widehat{f}(x_0)^2 + \mathbb{E}[(\widehat{y} - \widehat{f}(x_0))^2]$$  

Which can be further factorized to:  

$$= \mathbb{E}[(y_0 - \mathbb{E}[y_0])^2] + (\mathbb{E}[y_0] - \widehat{f}(x_0))^2
+ \mathbb{E}[(\widehat{y}_0 - \widehat{f}(x_0))^2]$$  

Because \\( \widehat{y}_0 - \widehat{f}(x_0) = \epsilon \\), we can rewrite the
  third term:  

$$= \color{blue}{\mathbb{E}[(y_0 - \mathbb{E}[y_0])^2} + \color{green}{(\mathbb{E}[y_0]
    - \widehat{f}(x_0))^2} + \color{red}{\mathbb{E}[\epsilon^2]}$$  

Also, since \\( \widehat{y}_0 = \widehat{f}(x_0) + \epsilon \\) and
  \\( \mathbb{E}[\epsilon] = 0 \\), this is exactly the same as:  

$$\mathbb{E}[(y_0 - \widehat{f}(x_0))^2]= \color{blue}{Var(\widehat{f}(x_0))}
+ \color{green}{[Bias(\widehat{f}(x_0))]^2} + \color{red}{Var(\epsilon)}$$  

In order to minimize the expected test error, we need to select a statistical learning
  method that simultaneously achieves *low variance* and *low bias*. Since variance
  and squared bias are nonnegative, expected test MSE can never lie below
  \\( Var(\epsilon) \\).  

*Variance* refers to the amount by which \\( \widehat{f} \\) would change if we
  estimated it using a different training set. Ideally, the estimate for \\( f \\)
  should not vary much between training sets, but if a method has high variance,
  then small changes in the training data can result in large changes of
  \\( \widehat{f} \\). In general, more flexible models have higher variance.  

*Bias* refers to the error that is introduced by approximating a real-life problem,
  which may be extremely complicated, by a much simpler model. In general, more flexible
  models result in less bias.  

Good test set performance of a statistical method requires low variance as well
  as low squared bias.

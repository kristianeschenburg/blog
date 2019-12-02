---
title: "Relationship Between Poisson and Multinomial"
layout: post
date: 2018-11-08 01:12:32
math: true
published: true
mathjax: true
paginate_path: "/Posts/page:num/"
---

In this post, I'm going to briefly cover the relationship between the Poisson distribution and the Multinomial distribution.  

Let's say that we have a set of independent, Poisson-distributed random variables $Y\_{1}, Y\_{2}... Y\_{k}$ with rate parameters $\lambda\_{1}, \lambda\_{2}, ...\lambda\_{k}$.  We can model the sum of these random variables as a new random variable $N = \sum\_{i=1}^{k} Y\_{i}$.

<!--more-->

Let start with $k=2$.  We can define the distrbution of $F\_{N}(n)$ as follows:

$$
\begin{align}
&= P(N \leq n) \\\\\\
&= P(Y\_{1} + Y\_{2} \leq n) \\\\\\
&= P(Y\_{1} = y\_{1}, Y\_{2} = n - y\_{1}) \\\\\\
&= P(Y\_{1} = y\_{1}) \cdot P(Y\_{2} = n-y\_{1}) \\\\\\
&= \sum\_{y\_{1}=0}^{n} \frac{e^{-\lambda\_{1}}\lambda\_{1}^{y\_{1}}}{y\_{1}!} \cdot \frac{e^{-\lambda\_{2}}\lambda\_{2}^{n-y\_{1}}}{(n-y\_{1})!} \\\\\\
&= e^{-(\lambda\_{1}+\lambda\_{2})} \sum\_{y\_{1}=0}^{n} \frac{\lambda\_{1}^{y\_{1}}\lambda\_{2}^{n-y\_{1}}}{y\_{1}!(n-y\_{1})!} \\\\\\
&= e^{-(\lambda\_{1}+\lambda\_{2})} \sum\_{y\_{1}=0}^{n} \frac{n!}{n!}\frac{\lambda\_{1}^{y\_{1}}\lambda\_{2}^{n-y\_{1}}}{y\_{1}!(n-y\_{1})!} \\\\\\
&= \frac{e^{-(\lambda\_{1}+\lambda\_{2})}}{n!} \sum\_{y\_{1}=0}^{n} {n\choose y\_{1}} \lambda\_{1}^{y\_{1}}\lambda\_{2}^{n-y_{1}} \\\\\\
\end{align}
$$

Here, we can apply the Binomial Theorem to the summation to get the following (remember that the Binomial Theorem says, for two numbers $x$ and $y$, that $(x+y)^{n} = \sum\_{i=0}^{n} {n \choose i}x^{i}y^{n-i}$):

$$
\begin{align}
\frac{e^{-(\lambda\_{1}+\lambda\_{2})}(\lambda\_{1} + \lambda\_{2})^{n}}{n!} \\\\\\
\end{align}
$$

which we see is in fact just another Poisson distribution with rate parameter equal to $\lambda\_{1} + \lambda\_{2}$.  This shows that the sum of independent Poisson distributed random variables is also a Poisson random variable, with rate parameter equal to the sum of the univariate rates.  By induction, we see that for $k$ independent Poisson distributed random variables $Y\_{1}...Y\_{k}$, their sum $\sum\_{i=1}^{k} Y\_{i} \sim Poisson(\sum\_{i=1}^{k} \lambda\_{i})$.

Now let's say we're interested in modeling the conditional distribution of $(Y\_{1}...Y\_{k}) \mid \sum\_{i=1}^{k} = n$.  By definition of conditional probability, we have that

$$
\begin{align}
P(\bar{Y} \mid N=n) &= \frac{P(\bar{Y} \; \cap \; N=n)}{P(N=n)} \\\\\\
&= \frac{P(\bar{Y})}{P(N=n)} \\\\\\
\end{align}
$$

We have the following:

$$
\begin{align}
P(\bar{Y} \mid N=n) &= \frac{P(\bar{Y} \; \cap \; N=n)}{P(N=n)} \\\\\\
&= \Big( \prod\_{i=1}^{k} \frac{e^{-\lambda\_{i}} \cdot \lambda\_{i}^{y\_{i}}}{y\_{i}!} \Big) \Big/ \frac{e^{-\sum\_{i=1}^{k} \lambda\_{i}}(\sum\_{i}^{k} \lambda\_{i})^{n}}{n!} \\\\\\
&= \Big( \frac{ e^{-\sum\_{i=1}^{k}} \prod\_{i=1}^{k} \lambda\_{i}^{y\_{i}}}{\prod\_{i=1}^{k} y\_{i}!} \Big) \Big/ \frac{e^{-\sum\_{i=1}^{k} \lambda\_{i}}(\sum\_{i}^{k} \lambda\_{i})^{n}}{n!} \\\\\\
&= { n \choose y\_{1}, y\_{2}, ...y\_{k}} \frac{\prod\_{i=1}^{k} \lambda\_{i}^{y\_{i}}} { \sum\_{i}^{k} \lambda\_{i})^{n}} \\\\\\
&= { n \choose y\_{1}, y\_{2}, ...y\_{k}}  \prod\_{i=1}^{k} \Big( \frac{ \lambda\_{i} }{\sum\_{i}^{k} \lambda\_{i}} \Big)^{y\_{i}} \\\\\\
&\sim MultiNom(n; \frac{\lambda\_{1}}{\sum\_{i=1}^{k}}, \frac{\lambda\_{2}}{\sum\_{i=1}^{k}}, ... \frac{\lambda\_{k}}{\sum\_{i=1}^{k}}) \\\\\\
\end{align}
$$

So finally, we see that, given the sum of independent Poisson random variables, that conditional distribution of each element of the Poisson vector is Multinomial distributed, with count probabilities scaled by the sum of the individual rates.  Importantly, we can extend these ideas (specifically the sum of independent Poisson random variables) to other models, such as splitting and merging homogenous and non-homogenous Poisson Point Processes.

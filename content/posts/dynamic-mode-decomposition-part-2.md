---
title: "Exploring Dynamical Systems With DMD: Part 2"
date: 2018-05-24 11:30:43
mathjax: true
show_reading_time: true
---

In my previous post on [Dynamic Mode Decomposition]({% post_url 2018-05-24-dynamic-mode-decomposition-part-1 %}), I discussed the foundations of DMD as a means for linearizing a dynamical system.  In this post, I want to look at a way in which we can use rank-updates to incorporate new information into the spectral decomposition of our linear operator, $A$, in the event that we are generating online measurements from our dynamical system.  If you want a more-detailed overview of this topic, [Zhang 2017](https://arxiv.org/abs/1707.02876) developed the theory, along with open source code, for testing this method.

<!--more-->

Recall that we are given an initial data matrix

$$
\begin{align}
X = \begin{bmatrix}
x\_{n\_{1},m\_{1}} & x\_{n\_{1},m\_{2}} & x\_{n\_{1},m\_{3}} & ... \\\\\\
x\_{n\_{2},m\_{1}} & x\_{n\_{1},m\_{2}} & x\_{n\_{2},m\_{3}} & ... \\\\\\
x\_{n\_{3},m\_{1}} & x\_{n\_{1},m\_{2}} & x\_{n\_{3},m\_{3}} & ... \\\\\\
... & ... & ... & ... \\\\\\
\end{bmatrix}
\in R^{n \times m}
\end{align}
$$

which we can split into two matrices, shifted one unit in time apart:

$$
\begin{align}
X^{\ast} &=
\begin{bmatrix}
\vert & \vert & & \vert \\\\\\
\vec{x}\_1 & \vec{x}\_2  & \dots & \vec{x}\_{m-1}  \\\\\\
\vert & \vert & & \vert \\\\\\
\end{bmatrix} \in R^{n \times (m-1)} \\\\\\
Y &= \begin{bmatrix}
\vert & \vert & & \vert \\\\\\
\vec{x}\_2 & \vec{x}\_3  & \dots & \vec{x}\_{m}  \\\\\\
\vert & \vert & & \vert \\\\\\
\end{bmatrix} \in R^{n \times (m-1)}
\end{align}
$$

and we are interested in solving for the linear operator $A$, such that

$$
\begin{align}
Y = AX^{\ast}
\end{align}
$$

For simplicity, since we are no longer using the full matrix, I'll just refer to $X^{\ast}$ as $X$.  In the previous post, we made the constraint that $n > m$, and that rank($X$) $\leq m < n$.  Here, however, we'll reverse this assumption, and such that $m > n$, and that rank($X$) $\leq m < n$, such that $XX^{T}$ is invertible, so by multiplying both sides by $X^{T}$ we have

$$
\begin{align}
AXX^{T} &= YX^{T} \\\\\\
A &= YX^{T}(XX^{T})^{-1} \\\\\\
A &= QP\_{x} \\\\\\
\end{align}
$$

where $Q = YX^{T}$ and $P\_{x} = (XX^{T})^{-1}$.  Now, let's say you observe some new data $x\_{m+1}, y\_{m+1}$, and you want to incorporate this new data into your $A$ matrix.  As in the previous post on [Rank-One Updates]({% post_url 2018-05-11-rank-one-updates %}), we saw that directly computing the inverse could potentially be costly, so we want to refrain from doing that if possible.  Instead, we'll use the Shermann-Morrison-Woodbury theorem again to incorporate our new $x\_{m+1}$ sample into our inverse matrix, just as before:

$$
\begin{align}
(X\_{m+1}X^{T}\_{m+1})^{-1} = P\_{x} + \frac{P\_{x}x\_{m+1}x\_{m+1}^{T}P\_{x}}{1 + x\_{m+1}^{T}P\_{x}x\_{m+1}}
\end{align}
$$

Likewise, since we're appending new data to our $$Y$$ and $$X$$ matrices, we also have

$$
\begin{align}
Y\_{m+1} = \begin{bmatrix}
Y & y\_{m+1} \end{bmatrix} \\\\\\
X\_{m+1} = \begin{bmatrix} \\\\\\
X & x\_{m+1} \end{bmatrix} \\\\\\
\end{align}
$$

such that

$$
\begin{align}
Y\_{m+1} X\_{m+1}^{T} &= YX^{T} + y\_{m+1}x\_{m+1}^{T} \\\\\\
&= Q + y\_{m+1}x\_{m+1}^{T} \\\\\\
\end{align}
$$

which is simply the sum of our original matrix $Q$, plus a rank-one matrix.  [Zhang 2017](https://arxiv.org/abs/1707.02876) go on to describe some pretty cool "local" DMD schemes, by incorporating weights, as well as binary thresholds, that are time-dependent into the computation of the linear operator, $A$.
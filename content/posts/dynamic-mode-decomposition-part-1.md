---
title: "Exploring Dynamical Systems With DMD: Part 1"
date: 2018-05-22 14:02:52
mathjax: true
show_reading_time: true
---

In the next two posts, I want to talk briefly about an algorithm called Dynamic Mode Decomposition (DMD).  DMD is a spatiotemporal modal decomposition technique that can be used to identify spatial patterns in a signal (modes), along with the time course of these spatial patterns (dynamics).  As such, the algorithm assumes that the input data has a both a spatial and a temporal component.  We are interested in modeling *how* the system evolves over time.  

<!--more-->

If you'd like to find more information about DMD, [Schmid 2010](https://hal-polytechnique.archives-ouvertes.fr/hal-01020654/document) a good reference.  Likewise, if you'd like to follow along with the code for the following analysis, see [my repo](https://github.com/kristianeschenburg/dmd).  For a more in-depth analysis that applies DMD to brain activity in the resting brain, see our recently-published [paper](https://www.frontiersin.org/articles/10.3389/fncom.2019.00075/full) my colleagues and I wrote, along with the [code](https://github.com/kunert/DMD_RSN) used for our analysis.

## The DMD Algorithm

Let's assume that you've taken $n$ measurements from specific points in space for $m$ time points, where for now we assume that $m\lt n$.  For now, we'll assume that the sampling frequency, $\omega$, is stable across the entire experiment.  We define our entire data matrix as

$$
\begin{align}
X = \begin{bmatrix}
x\_{n\_{1}, m\_{1}} & x\_{n\_{1}, m\_{2}} & x\_{n\_{1}, m\_{3}} & ... \\\\\\
x\_{n\_{2}, m\_{1}} & x\_{n\_{1}, m\_{2}} & x\_{n\_{2}, m\_{3}} & ... \\\\\\
x\_{n\_{3}, m\_{1}} & x\_{n\_{1}, m\_{2}} & x\_{n\_{3}, m\_{3}} & ... \\\\\\
... & ... & ... & ... \\\\\\
\end{bmatrix}
\in R^{n \times m}
\end{align}
$$

We are interested in solving for the matrix, $A \in R^{n \times n}$, such that

$$
\begin{align}
x\_{t+1} = A x\_{t} \; \; \forall \; \; t = 1,2,...m-1 \\\\\\
\end{align}
$$

Given our full data matrix $X$, we can define two matrices $X^{\ast}$ and $Y$ such that

$$
\begin{align}
X^{\ast} &= \begin{bmatrix}
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

so that we can write

$$
\begin{align}
Y = AX^{\ast}
\end{align}
$$

If $n$ is small, this is relatively easy to compute -- however, if $n$ is large, as is the case when modeling temporal dynamics in resting-state MRI, it would be computationally inefficient to compute A directly.  To alleviate this, we can make use of the Singular Value Decomposition (SVD) of our predictor matrix $X^{\ast}$.  We define the SVD of $X^{\ast}$ as $X^{\ast} = U \Sigma V^{T}$ as well as the Moore-Penrose psuedo-inverse of $X^{\ast} = X^{\dagger}$ as $
X^{\dagger} = V \Sigma^{-1} U^{T}$

We can then write

$$
\begin{align}
YX^{\dagger}  = YV \Sigma^{-1} U^{T} = A X^{\ast}X^{\dagger} = A \\\\\\
\end{align}
$$

Additionally, if we assume that rank$(X^{\ast}) = r \leq m$, then we can use the truncated SVD such that $U \in R^{n \times r}$ and $V^{T} \in R^{r \times m}$

and

$$
\begin{align}
\Sigma = \begin{bmatrix}
\sigma\_{1} & 0 & 0 & ... \\\\\\
0 & \sigma\_{2} & 0 & ... \\\\\\
0 & 0 & \ddots & ... \\\\\\
\vdots & \vdots & \vdots & \sigma\_{r} \\\\\\
\end{bmatrix} \in R^{r \times r} \\\\\\
\end{align}
$$

As it stands now, we still compute an $A \in R^{n \times n}$ matrix.  However, because we have a potentially low-rank system, we can apply a Similarity Transformation to $A$ in order to reduce its dimensionality, without changing its spectrum.  Using our spatial singular vectors $U$, we define

$$\begin{align}
\tilde{A} &= U^{T} A U \\\\\\
&= U^{T} (YV \Sigma^{-1} U^{T}) U \\\\\\
&= U^{T} Y V \Sigma^{-1} \\\\\\
\end{align}$$

where $\tilde{A} \in R^{r \times r}$.  If we consider the above SVD, we see that $U$ is the matrix of left singular vectors, an orthogonal basis that spans $C(X^{\ast})$, which is an r-dimensional subspace of $R^{n}$.  Thus, the similarity transform represents a mapping $f(A) = U^{T} A U : R^{n} \rightarrow R^{r}$.  We now have a reduced-dimensional representation of our linear operator, from which we can compute the spatial modes and dynamic behavior of each mode.  First, however, because of the notion of variance captured by the singular values of our original predictor matrix, we weight $\tilde{A}$ by the singular values as

$$
\begin{align}
\hat{A} = \Sigma^{\frac{1}{2}} \tilde{A} \Sigma^{\frac{1}{2}} \\\\\\
\end{align}
$$

such that our computed spatial modes have been weighted by the amount they contribute to our measured signal.  We can now compute the eigendecomposition of $\hat{A}$ as

$$
\begin{align}
\hat{A} W = W \Lambda \\\\\\
\end{align}
$$

where the eigenvectors $W$ are the reduced-dimension representations of our spatial modes, and the eigenvalues $\Lambda$ capture the dynamic behavior of our spatial modes.  Because our original data matrix $X^{\ast}$ had spatial dimension $n$ and our eigenvectors have dimension $r$, we need to up-project our eigenvectors $W$ to compute the final spatial modes, via

$$
\begin{align}
\Phi = Y V \Sigma^{\frac{-1}{2}}W \\\\\\
\end{align}
$$

From the SVD of our prediction matrix $X^\ast=U \Sigma V^{T}$, the matrix $V \in R^{m \times r}$ is the matrix of right singular vectors, an orthogonal basis spanning the space of $X^{\ast T}$ (i.e. $r$ basis vectors spanning the space of the measured time courses).  Thus, we see that $H = (V \Sigma^{\frac{-1}{2}})W$ represents a linear combination of the temporal basis vectors (a mapping from $R^{r} \rightarrow R^{m}$) for each eigenvector $w\_{i}$ of $W$, weighted by the corresponding singular value $\sigma\_{i}^{\frac{-1}{2}}$ (that acts to normalize the spatial mode amplitudes).  Finally, we see that $\Phi = X^{\ast}H$ computes how much of each temporal basis vector is present in the measured time course at each point in space.

Because we are modeling a dynamical system, we can compute the continuous time dynamics of our system using our spatial modes and eigenvalues as

$$
\begin{align}
\vec{x}(t) \approx \sum\_{i=1}^{r} b\_{i}\exp^{((\gamma\_{i} + 2i\pi f\_{i})\cdot t)} \vec{\phi}\_{i} \\\\\\
\end{align}
$$

where $\gamma\_{i}$ is a growth-decay constant and $f\_{i}$ is the frequency of oscillation of the spatial mode $\phi\_{i}$.  We can compute these two constants as

$$
\begin{align}
\gamma\_{i} &= \frac{\text{real}(\text{ln}(\lambda\_{i}))}{\Delta t} \\\\\\
f\_{i} &= \frac{\text{imag}(\text{ln}(\lambda\_{i}))}{2\pi \Delta t} \\\\\\
\end{align}
$$

So, we can see that DMD linearizes our measured time series, by fitting what can be analogized to a "global" regression.  That is, instead of computing how a single time point predicts the next time point, which could readily be solved using the simple [Normal equations](https://en.wikipedia.org/wiki/Linear_least_squares#Derivation_of_the_normal_equations), DMD computes how a matrix of time points predicts another matrix of time points that is shifted one unit of time into the future.  To this extent, DMD minimizes the Frobenius norm of

$$
\begin{align}
\min \limits_{A} \lVert Y - AX^{\ast} \rVert^{2}\_{F} \\
\end{align}
$$

However, rather than explicitly computing the matrix $A$, DMD computes the eigenvectors and eigenvalues of $A$, by utilizing the [Singular Value Decomposition](https://en.wikipedia.org/wiki/Singular_value_decomposition), along with a [Similarity Transformation](https://en.wikipedia.org/wiki/Matrix_similarity), in order to generate a reduced-dimensional representation of $$A$$.

This spectral decomposition of our linear operator is of particular importance, because it sheds light on the fact the DMD models the temporal dynamics of our system using a [Fourier basis](https://en.wikipedia.org/wiki/Fourier_series).  Each spatial mode is represented by a particular Fourier frequency along and growth-decay constant that determines the future behavior of our spatial mode.  Additionally, the Fourier basis also determines what sorts of time series can be modeled using DMD -- time series that are expected to have sinusoidal behavior will be more reliably modeled using DMD, whereas signals that show abrupt spike patterns might be more difficult to model.
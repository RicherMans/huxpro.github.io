---
layout: post
title: Microphone Array Signal Processing - Introduction
---

A short introduction to microphone array signal processing, directly taken from the book [Microphone Array Signal Processing](http://link.springer.com/content/pdf/10.1007/978-3-540-78612-2.pdf%5Cnhttp://www.springerlink.com/content/r0402k/?p=05de05ef4a864072a54e82522c1bf267&pi=0).

# Stationary Signal Filtering 

Here we briefly discuss two filter techniques for stationary (distribution independent of time) signals. 

## Wiener Filter

Consider a zero-mean clean speech signal $x(k)$, contaminated by a zero-mean process $v(k)$, so that the observed signal $y(k)$ at time $k$ is:

$$y(k)  = x(k) + v(k)$$

Assuming all signals are stationary, the objective is to retrieve $x(k)$ from $y(k)$. The Wiener filter uses a finite response (FIR) filter of length $L$ (equal to the visible output signals length), defined as:

$$ h = [h_0,h_1,\ldots, h_{L-1}]^T $$

This filter is applied onto the observed signal $y(k)$ with the goal of removing $v(k)$ from $y(k)$, hence the potential error of this filter would be:

$$ 
\begin{align}
e(k) &= x(k) - h^T y(k) \\
&= x(k) - z(k) \\
z(k) &= h^T y(k) 
\end{align}
$$

After defining the error, a criterion can be formulated, e.g., Mean square error (MSE):

$$
\begin{align*}
\mathcal{L}(h) &= E[e^2(k)] \\
&= E[(x(k)-z(k))^2] \\
&= E[x(k)^2 - 2x(k)z(k) + z(k)^2] \\
&= E[x(k)^2] - 2 E[x(k)z(k)] + E[h^Ty(k)y(k)^Th] \\
&= E[x(k)^2] - 2 E[x(k)y(k)] h + h^T E[y(k)y(k)^T] h\\
&= \sigma_x^2 - 2 r_{yx}^T h +  h^TR_{yy}
\end{align*}
$$

Therefore we can specify a value for $\hat{h}$ the minimization problem as, by using the matrix cookbook's definitions:

$$
\begin{align}
\frac{\delta x^T B x}{\delta x} &= (B + B^T) x\\
\frac{\delta x^T a}{\delta x} &= \frac{\delta a^T x}{\delta x} = a
\end{align}
$$

We obtain the required estimate for $\hat{h}$:

$$
\begin{align*}
\hat{h} &= \underset{h}{\operatorname{argmin}} \mathcal{L}(h) \\
\mathcal{L}(h)^{`} &= (R_{yy} + R_{yy}^T)h - r_{yx} \\
R_{yy}^{-1} r_{yx} &= h \\
&= R_{yy}^{-1} r_{yx}
\end{align*}
$$

But here lies a problem, since in general the original signal $x(k)$ is unobtainable, this formula cannot be used. However, one can refactor the expression $r_{yx}$ as:

$$
\begin{align}
r_{yx} &= E[y(k)x(k)] = E[y(k)[y(k)-v(k)]] = E[y(k)-y(k)] - E[v(k)v(k)] \\
&= r_{yy} - r{vv}
\end{align}
$$

With this simplification, the whole optimization problem only depends on the (maybe unknown ) noise $v(k)$ and the observation $y(k)$.

Therefore, optimal estimation in the wiener sense is:

$$
z(k) = \hat{h}y(k) = y(k) - r_{vv}^T R_{yy}^{-1} y(k)
$$

If we consider a particular filter, that does no noise reduction $h_1$ as:

$$\
\begin{align*}
h_1 &= [1,0, \ldots, 0]^T\\
|h_1| &= L
\end{align*}
$$

The corresponding MSE is then:

$$
\begin{align*}
\mathcal{L}(h_1) &= E[{x(k) - h_1^Ty(k) }^2]\\
&= E[{x(k) - y(k)}^2]\\
&= E[v^2(k)] = \omega_v^2
\end{align*}
$$

The created filter $h_1$ is equal to $h_1 = R_{yy}^{-1}r_{yy}$, since $R_{yy}^{-1}r_{yy} = E[(yy)^-1]E[yy] = 1$.

Another approach using this identity above, leads to the following filter:

$$
\hat{h} = h_1 - R_{yy}^{-1} r_{vv}\\
= [\frac{I}{SNR} + \tilde{R_vv}^{-1}\tilde{R_xx}]^{-1} \tilde{R_vv}^{-1}\tilde{R_xx}h_1
$$

The optimal estimation of this filter is therefore:

$$
\begin{align*}
z_w(k) &= \hat{h}^T y(k)\\
&= y(k) - r_{vv}^TR_{yy}^{-1}y(k)\\
E[z_w(k)] &= \hat{h}^TR_{xx}\hat{h} + \hat{h}^TR_{vv}\hat{h}
\end{align*}
$$

Even though this filter can improve the signal to noise ratio (SNR), it does not maximize it. 

## Frost Filter

The previously shortly described Wiener filter is, even though mathematically concise, in many cases not applicable for real world applications, since the error $e(k)$ is defined with regards to the reference $x(k)$, which often does not exist.

Therefore the criterion becomes:

$$
\mathcal{L}(h) = h^T R_{yy} h
$$

Minimization of this criterion leads to the obvious solution $h=0$. Fortunately in many applications, the filter $h$ is constrained by the following form:

$$
C^Th = u
$$

The constrained optimization problem can then be solved using lagrange multipliers.

$$
\operatorname{min} \mathcal{L}(h) \text{ subject to } C^Th = u\\
h_{F} = R_{yy}^{-1} C(C^TR_{yy}^{-1}C)^{-1}u
$$

It is important that $C$ is fullrank and $R_{yy}$ is invertible.

Both of these filters however, can only be used for stationary signals (e.g., distribution does not change over time), which is a strict constraint, unseen in real world scenarios.


# Non-Stationary Signal Filtering 

In case of non-stationary signals, a possible approach for modelling was intruduced by Rudolf E. Kálmán. This approach models an observed signal as follows:

$$
\begin{align*}
y(k) &= x(k) + v(k)\\
&= h_1^Tx(k) + v(k)
\end{align*}
$$

Where $h_1$ is a one-hot encoded vector (zeros with a single one). In this model, the vector $h_1$ also referred to as state vector, does change it's values depending on the timestep $k$. The kalman filter seems to be similar in formulation and implementation as a two state hidden markov model. 
The optimization of the Kalman filter done with a two step (similar to HMM).

We first define the speech signals models:

$$
\begin{align*}
x(k) &= \sum_{l=1}^L a_lx(k-l) + v_x(k)\\
&= \sum_{l=1}^{L-1} a_lx(k-l) + v_x(k) h_1\\
&= Ax(k-1) + v_x(k) h_1
\end{align*}
$$

With the transition matrix $A$ defined as:

$$
A=
\begin{bmatrix}
a_1 & a_2 & \ldots & a_{L-1} & a_{L}\\
1 & 0 & \ldots & 0 & 0 \\
0 & 1 & \ldots & 0 & 0 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & \ldots & 1 & 0 
\end{bmatrix}
$$

Therefore, the objective is to find the optimal linear MSE of $x(k)$, for the two given objective functions:

$$
\begin{align*}
x(k) &= Ax(k-1) + v_x (k) h_1 \\
y(k) &= h_1^T x(k) + v(k)
\end{align*}
$$


* State Equation 
 
$$
x(k) = Ax(k - 1) + v_x(k)h_1
$$

* Observation Equation

$$
y(k) = h_1^Tx(k) + v(k)
$$

* Initialization

$$
\begin{align*}
\hat{x}(0|0) &= E[x(0)]\\
R_{ee}(0|0) &= E[x(0)x^T(0)]
\end{align*}
$$

* Computation

$$
\begin{align*}
\hat{x}(k | k -1) &= A\hat{x} (k-1|k-1)\\
R_{ee}(k | k-1) &= AR_{ee}\\
k(k) &= \frac{R_{ee}(k | k-1)h_1}{h_1^TR_{ee} (k | k-1)h_1 + \sigma_v^2(k)}\\
\hat{x}(k|k) &= \hat{x}(k|k-1) + k(k)[y(k - h_1^T\hat{x}(k|k-1)]\\
R_{ee}(k|k) &= [I - k(k)h_1^T]R_{ee}(k|k-1)\\
\end{align*}
$$

As it can be seen the kalman filter is recursively defined and starts from a beginning timestep. The matrices $R_{ee}$ needs to be previously initialized. Unfortunately the initialization of this matrix is essential to the effect, thus much research has been done to provide feasible starting estimates.

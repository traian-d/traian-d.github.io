---
layout: post
title:  "Cosines and Correlations"
date:   2020-03-23 18:17:00 +0100
---

This is a small note in which I would like to clarify an idea that I held intuitively, but that I never really understood rigorously: namely that correlations between random variables are in a certain way like cosines between vectors.

To do this I had to dig a bit into some simple concepts that I was nevertheless not fully aware of, mostly following Axler's *Linear Algebra Done Right* [1].

### Preliminaries

#### Inner Products and Norms

For a vector space *V* over a field **F** an inner product on V is a function that takes each ordered pair (u, v) of elements of V to a number $$\langle u, v \rangle \in \textbf{F}$$ with the following properties of interest:

**Positivity**

$\langle v, v \rangle \geq 0 \text{ for all } v \in V$.

**Definiteness**

$\langle v, v \rangle = 0 \Leftrightarrow v = 0$.

**Additivity**

$\langle u + v, w \rangle = \langle u, w \rangle + \langle v, w \rangle \text{ for all } u, v, w \in V$.

$\langle u, v + w\rangle = \langle u, v \rangle + \langle u, w \rangle \text{ for all } u, v, w \in V$.

**Homogeneity in the first slot**

$\langle \lambda u, v \rangle = \lambda\langle u, v \rangle \text{ for all } \lambda \in \textbf{F}; u, v \in V$.

An **inner product space** is a vector space V along with an inner product on V.

For the standard case of two vectors $x, y \in \mathbb{R}^n$ the inner product is the familiar **dot product** given by:

$x \cdot y = x_{1}y_{1} + ... + x_{n}y_{n}$ where $x = (x_{1}, ..., x_{n})$ and $y = (y_{1}, ..., y_{n})$.

A **norm** is defined by: $\|\|v\|\| = \sqrt{\langle v, v \rangle}$ for $v \in V$ and has the following properties of interest:
 
 * $\|\|v\|\| = 0 \Leftrightarrow v = 0.$
 
 * $\|\| \lambda v \|\| = \| \lambda \| \|\|v\|\| \text{ for all } \lambda \in \textbf{F}.$

 For the standard case of $x = (x_{1}, ..., x_{n}) \in \mathbb{R}^{n}$ this will simply be the Euclidean norm $\|\|x\|\| = \sqrt{x_1^{2} + ... + x_n^{2}}$.

#### Orthogonal decomposition

The usual concept of orthogonality can be generalized for two vectors $u, v \in V$ by saying that they are orthogonal to each other if their inner product is 0.

For $u, v \in V, c = \frac{\langle u, v \rangle}{\|\|v\|\|^2}, w = u - \frac{\langle u, v \rangle}{\|\|v\|\|^2}v$ we can write $u = cv + w$. In this case it can be shown that $\langle v, w \rangle = 0$. This is a useful trick that allows you to break any vector up into a sum of two orthogonal vectors.

#### Cauchy–Schwarz Inequality

For $u, v \in V$ we have $\|\langle u, v\rangle\| \leq \|\|u\|\|\|\|v\|\|$.


### Cosine and the Inner Product

For a triangle made up of the vectors $u, v, u-v \in \mathbb{R}^n$ and $\theta$ the angle between u and v, we can write the law of cosines:

$$
\begin{align*}
\langle u - v, u - v\rangle &= \langle u, u\rangle + \langle v, v\rangle - 2\|u\|\|v\|cos\theta \Leftrightarrow\\
\langle u, u\rangle - \langle u, v\rangle - \langle v, u\rangle + \langle v, v\rangle &= \langle u, u\rangle + \langle v, v\rangle - 2\|u\|\|v\|cos\theta \Leftrightarrow\\
 - \langle u, v\rangle - \langle v, u\rangle &= - 2\|u\|\|v\|cos\theta \Leftrightarrow\\
 \langle u, v\rangle &= \|u\|\|v\|cos\theta
\end{align*}
$$

The last line works because for $\mathbb{R}^n$ the inner product is symmetrical. We can also rewrite it as:

$$
cos\theta = \frac{\langle u, v\rangle}{\|u\|\|v\|} 
$$

### Cosines and Correlations

Already from the way we can rewrite the cosine of $\theta$ you sense a resemblance with the Pearson correlation coefficient. More than than, for random variables it can be shown that the set of real-valued random variables of finite variance forms an inner product space with the operation $$\langle X, Y \rangle := \mathbb{E}[X Y]$$. This means that for two random variables we can write the correlation coefficient as:

$$
\begin{align*}
\rho_{X, Y} &= \frac{\mathbb{E}[(X - \mu_X)(Y - \mu_Y)]}{\sqrt{\mathbb{E}[(X - \mu_X)^2]\mathbb{E}[(Y - \mu_Y)^2]}} \\
			&= \frac{\langle X - \mu_X, Y - \mu_X \rangle}{\|X - \mu_X\|\|Y - \mu_Y\|}
\end{align*}
$$

If we consider mean-centered variables ($$\mu_X, \mu_Y = 0$$), this is essentially the same expression as the one used for $$cos\theta$$. At first sight it's quite tempting to consider them exactly equal. However this seems to me naive and actually incorrect. Although both correlations and cosines lie in [-1, 1] where a value of 0 represents orthogonality, on closer inspection the equivalence breaks down. For one, we would expect that $$\rho_{X, Y} = 0.5 = cos 45^\circ$$. It's easy to check that this is false, since $$cos 45^\circ = 0.5253$$. If we take a look at the [arccos](https://en.wikipedia.org/wiki/Inverse_trigonometric_functions#/media/File:Arcsine_Arccosine.svg) function it's plainly clear that it's not linear on [-1, -0.5] and [0.5, 1]. 

On top of this, of course, the two functions operate on completely different vector spaces (real/complex vector space for cos, a Hilbert space of random variables for the correlation) that can be treated as analogous for some purposes, but that are not identical.


### An application: the case of three random variables

I find it interesting to think about the following setup: consider X, Y, Z r.v.'s with correlations $$\rho_{X, Y}, \rho_{X, Z}, \rho_{Y, Z}$$. Let's say we fix $$\rho_{X, Y}$$ and $$\rho_{X, Z}$$, what values can $$\rho_{Y, Z}$$ take?

To make the math work nicely we can impose the further restriction of unit variance (leads to e.g. $$\rho_{X, Y} = \langle Y, X \rangle$$ and $$\|X\|^2 = 1$$). This shouldn't affect the generality of the results, since standardization doesn't affect correlations/cosines.

Following [2], a first step would be to do an orthogonal decomposition of Y and Z:

$$
\begin{align*}
Y &= \frac{\langle Y, X \rangle}{\|X\|^2} X + Y - \frac{\langle Y, X \rangle}{\|X\|^2} X\\
  &= \rho_{X, Y} X + Y - \rho_{X, Y} X\\
  &\quad \quad \quad and\\
Z &= \rho_{X, Z} X + Z - \rho_{X, Z} X
\end{align*}
$$

then we have:

$$
\begin{align*}
\langle Y, Z \rangle &= \langle \rho_{X, Y} X + (Y - \rho_{X, Y} X), \rho_{X, Z} X + (Z - \rho_{X, Z} X) \rangle\\
					 &= \langle \rho_{X, Y} X, \rho_{X, Z} X\rangle + \\	
					 &\quad \langle \rho_{X, Y} X, (Z - \rho_{X, Z} X) \rangle + \langle (Y - \rho_{X, Y} X), \rho_{X, Z} X \rangle + \\
					 &\quad \langle (Y - \rho_{X, Y} X), (Z - \rho_{X, Z} X) \rangle\\
					 &= \rho_{X, Y}\rho_{X, Z} + \langle Y - \rho_{X, Y} X, Z - \rho_{X, Z} X \rangle
\end{align*}
$$

The only thing we need to clear up is exactly what the limits of the second term might be. For this we can apply the Cauchy-Schwarz inequality to it:

$$
|\langle Y - \rho_{X, Y} X, Z - \rho_{X, Z} X \rangle| \leq \|Y - \rho_{X, Y} X\|\|Z - \rho_{X, Z} X\|
$$

Developing the first term on the right side gives:

$$
\begin{align*}
\sqrt{\langle Y - \rho_{X, Y} X, Y - \rho_{X, Y} X \rangle} &= \sqrt{1 - \rho_{X, Y}\langle Y, X\rangle - \rho_{X, Y}\langle X, Y\rangle + \rho_{X, Y}^2}\\
															&= \sqrt{1 - \rho_{X, Y}^2}
\end{align*}
$$

Finally we get:

$$
 \rho_{X, Y}\rho_{X, Z} - \sqrt{(1 - \rho_{X, Y}^2)(1 - \rho_{X, Z}^2)} \leq 
 \rho_{Y, Z} \leq 
 \rho_{X, Y}\rho_{X, Z} + \sqrt{(1 - \rho_{X, Y}^2)(1 - \rho_{X, Z}^2)}
$$

### References

[1] Sheldon Axler, *Linear Algebra Done Right, 3rd edition*, *Chapter 6.A*

[2] [Stack Exchange](https://math.stackexchange.com/questions/284877/correlation-between-three-variables-question)


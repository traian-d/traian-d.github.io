---
layout: post
title:  "Sampling from Correlated Random Variables"
date:   2020-04-17 22:59:00 +0100
---

Sampling from correlated random variables is part of a growing series of tasks that at first sight seemed trivial to me, but that on further inspection prove to be quite tricky. Since now I'm working towards a bigger post on the curse of dimensionality, one of the things I needed to do was to somehow sample from a bunch of correlated random variables. More so, what I wanted to have was a distribution with uniform marginals.

As it turns out, there's no very straight forward way to introduce a correlation structure in a multivariate uniform distribution. If you want something off the shelf it seems that you have a relatively limited choice, with the [Multivariate Normal](https://en.wikipedia.org/wiki/Multivariate_normal_distribution) being the main option available.

Luckily, I was already familiar with [copulas](https://en.wikipedia.org/wiki/Copula_(probability_theory)), which provide a flexible framework to create distributions with a given correlation structure. In this post I will introduce a few key definitions and results regarding copulas and I will show an easy way to draw from a copula using the [Copula](https://cran.r-project.org/web/packages/copula/copula.pdf) R package.

### Copulas

The nice thing about using copulas to model the dependence of multivariate distributions is that they allow you to decouple the marginals from their correlation structure. This can come in really handy if you want to model data that may not fit any of the (few) common multivariate distributions that have an analytical form. In such a situation you can first estimate one dimensional marginals that you subsequently combine using one of a range of known copula functions in order to obtain a joint multivariate CDF.

#### Definition

Most of the technical statements in this post were taken from [1], which I think is a really comprehensive and useful overview of copula theory. In particular I have taken the definition of a copula from *Definition 2.4*, as follows:

An n-dimensional copula $C$ is a function with domain $[0, 1]^n$ such that:

1. $C$ is grounded and n-increasing.

2. $C$ has marginals $C_k, k=1, 2, ..., n$ which satisfy $C_k(u) = u$ for all $u \in [0, 1]$.

What this essentially says is that a copula is a function that can map an n-dimensional unit cube to the unit interval. The function also has to be non-decreasing and right-continuous and should converge to 0 or 1, when all of its marginals tend towards 0 or 1 respectively. While more precise technical details are available in Section 2.1 of [1], the end goal of this mathematical grounding is to define an object that is essentially an n-dimensional joint CDF with uniform marginals.

#### Sklar's theorem

Sklar's theorem explains the relationship between multivariate CDF's and their univariate marginals and is the most important result in copula theory. Following *Theorem 2.2* from [1], we can say:

For $F_x$, an n-dimensional CDF with marginals $F_1, F_2, ..., F_n$, there exists a unique copula C such that for all $x \in \mathbb{R}^n$ we have:

$\quad \quad \quad F_x(x_1, ..., x_n) = C(F_1(x_1), ..., F_n(x_n))$.

An important corollary of the theorem is that we can write a copula C as follows:

$\quad \quad \quad C(u_1, ..., u_n) = F_x(F_1^{-1}(u_1), ..., F_n^{-1}(u_n))$ for any $u_1, ..., u_n \in [0, 1]^n$.

### Limitations of the correlation coefficient and Kendall's tau

While Pearson correlation adequately captures linear dependence and is usually appropriate for elliptically jointly distributed rv's, it does suffer from a number of drawbacks that make it not as useful to measure copula dependence:

1. $\rho = 0 \not\Leftrightarrow $ independence (see [here](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient#/media/File:Correlation_examples2.svg) for example).

2. $\rho$ is not invariant under non-linear changes of scale: $\rho(log X_1, log X_2) \neq \rho(X_1, X_2)$.

3. $\rho$ is sensitive to outliers.

A number of other measures of dependence are however available. Of particular interest will be [Kendall's tau](https://en.wikipedia.org/wiki/Kendall_rank_correlation_coefficient). For two random variables $(X, Y)$ that are jointly distributed and $(\tilde{X}, \tilde{Y})$ an independent copy of the same random variables, Kendall's tau is the probability of a concordant pair minus the probability of a discordant pair:

$$
	\begin{align*}
		\tau_{X, Y} &= \mathbb{E}[sign((X - \tilde{X})(Y - \tilde{Y}))]\\
					&= P((X - \tilde{X})(Y - \tilde{Y}) > 0) - P((X - \tilde{X})(Y - \tilde{Y}) < 0)
	\end{align*}
$$

From *Theorem 3.3* of [1], if X and Y are linked together by a copula C we will have:

$$
	\begin{align*}
		\tau_{X, Y} &= 4\int_0^1 \int_0^1 C(u, v)dC(u,v) - 1
	\end{align*}
$$


### The Archimedean Family

For $\phi :[0, 1] \rightarrow [0, \infty]$ a continuous, strictly decreasing, convex function with $\phi(0)=+\infty$, $\phi(1)=0$, a 2-dimensional Archimedean copula is a copula of the form:

$C(u, v) = \phi^{-1}(\phi(u) + \phi(v))$

*Theorem 6.2* from [1] provides some useful properties of Archimedean copulas, that make them more amenable to multivariate extensions:

1. They are symmetric: $C(u, v) = C(v, u) \ (\forall)\  u, v \in [0, 1]$
2. They are associative: $C(C(u, v), w) = C(u, C(v, w))\ (\forall)\ u, v, w \in [0, 1]$

For my sampling purposes I have opted for a Frank copula, which has the generator: 

$\phi(t) = ln\frac{e^{-\theta t} - 1}{e^{-\theta} - 1}, \theta \in \mathbb{R} \setminus \\{0\\}$

After a bit of computation this yields the copula expression:

$
C_{Frank}(u, v) = -\frac{1}{\theta}ln\left( 1 + \frac{(e^{-\theta u} - 1)(e^{-\theta v} - 1)}{e^{-\theta} - 1}\right)
$

For a two-dimensional Frank copula, *Table 2* of [3] also gives an analytical formula for Kendall's tau:

$
\tau_{Frank} = 1 + \frac{4(D_1(\theta) − 1)}{\theta}
$

where $D_1$ denotes the [Debye function](https://en.wikipedia.org/wiki/Debye_function) of order 1.

#### Exchangeable and nested Archimedean copulas

The two dimensional formulation from above can be extended to n dimensions via either exchangeable or nested copulas. Following *Section 2* of [2], an exchangeable copula will be defined by:

$
C(u_1, ..., u_n) = \phi^{-1}(\phi(u_1) + ... + \phi(u_n))
$

This structure is however noticeably limited by the fact that all the components have the same dependence between themselves. To overcome this limitation we can use the associativity of Archimedean copulas to build nested versions, as follows:

$
C(u_1, ..., u_n) = \phi^{-1}(\phi(u_1) + C(u_2, ..., u_n))
$

This structure can actually be extended to allow different generators for each level. From [2], for n = 3, such a fully nested structure would have the form:

$
C(u_1, u_2, u_3) = C(u_1, C(u_2, u_3; \phi_1); \phi_0) = \phi_0^{-1}(\phi_0(u_1) + \phi_0(\phi_1^{-1}(\phi_1(u_2) + \phi_1(u_3)))
$

It's worth noting that exchangeable copulas can be combined with nested ones to give what are called partially nested copulas.

### Sampling from an Archimedean copula

Given a copula function C and U, V a pair of uniform rv's, we can generate observations from C by first drawing an observation *u* from U and noting that: 

$
c_u(v) = P(V \leq v | U = u) = \frac{\partial}{\partial u} C(u, v)
$

To this we can apply the probability integral transform to obtain $t = c_u^{-1}(v)$. This way the pair $(u, t)$ will actually fit the copula C.

Going through the necessary computations, for the Frank copula we will have:

$
t = -\frac{1}{\theta}ln\left(1 + \frac{v (e^{-\theta} - 1)}{v(1 - e^{-\theta u}) + e^{-\theta u}} \right)
$

With these analytical formulas figured out it's quite easy to now draw from a 2-dimensional Frank copula. I've opted to do everything in R:

```R
frank <- function(size, theta){
  u = runif(size)
  v = runif(size)
  t = (- 1 / theta) * log( (v * (exp(-theta)-1)) / (v*(1 - exp(-theta * u)) + exp(-theta*u)) + 1)
  
  return(list("u" = u, "v" = t))
}

d_1 <- function(t){
  return(t / (exp(t) - 1))
}

debye_1 <- function(x){
  # As it turns out this is already part of the copula package
  return((1 / x) * (integrate(d_1, 0, x)$value))
}

kendall_tau_frank <- function(theta){
  #1 + 4(D1(θ) − 1)/θ
  return(1 + 4 * (debye_1(theta) - 1) / theta)
}
```
Here are samples of n=20000 for different values of $\theta$ (the sample tau's have been calculated using the *Kendall* R package):

<div style="text-align:center;margin:20px 0px">
<img src="/assets/copula_sampling/frank_theta1.png" width="400" height="365">
</div>

<div style="text-align:center;margin:20px 0px">
<img src="/assets/copula_sampling/frank_theta10.png" width="400" height="365">
</div>

<div style="text-align:center;margin:20px 0px">
<img src="/assets/copula_sampling/frank_theta30.png" width="400" height="365">
</div>

The downside of taking the easy route for sampling is that things become more and more involved in higher dimensions. For that reason, [2] and [3] elaborate in significant detail other ways to sample both exchangeable and nested copulas. In particular I was really pleased to see that there's already a mature R package that implements Hofert's work, namely the *copula* package. To end I have therefore added a small example of how to sample from a nested Frank copula.

#### Sampling from a nested Frank copula

I've played around a little bit with different copula structures. It's interesting to note that simulation performance degrades very quickly if the outer $\theta$ parameters are set to values higher than the ones on a lower nesting level (e.g. in the following code setting *theta0* > *theta2*). This probably stems from the following fact observed in [3], p. 12:

*First note that for nested Archimedean copulas based on generators belonging to the same Archimedean family, all implemented families indeed lead to proper copulas if the generators on a more nested (inner) level have larger parameter values than the ones on a lower (outer) level. This is equivalent to saying that Kendall’s tau for a pair of random variables having a bivariate Archimedean copula which resides on a deeper nesting level as margin has to be larger than or equal to the one for a pair of random variables having an Archimedean marginal copula residing on a lower nesting level.*

Code and some plots follow.

```R
library('copula')

theta0 <- copClayton@iTau(0.2)
theta1 <- copClayton@iTau(0.75)
theta2 <- copClayton@iTau(0.9)

frank_copula <- onacopula("Frank", C(theta0, c(3,6), C(theta1, c(5,1), C(theta2, c(4,2)))))
# frank_copula <- onacopula("Frank", C(theta2, c(1, 2, 3, 4, 5, 6)))

set.seed(1)
start_time = Sys.time()

U9 <- rnacopula(500, frank_copula)
j <- allComp(frank_copula)
(vnames <- do.call(expression, lapply(j, function(i) substitute( U[I], list(I=0+i)))))

splom2(U9[, j], varnames= vnames, cex = 0.4, pscales = 0, title="some title")

end_time = Sys.time()
end_time - start_time
```

A 6 dimensional nested Frank copula with the structure *onacopula("Frank", C(theta0, c(3,6), C(theta1, c(5,1), C(theta2, c(4,2)))))*:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/copula_sampling/nested_frank_02_075_09.png" width="400" height="365">
</div>

A 6 dimensional exchangeable copula with $\theta = 0.9$:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/copula_sampling/exchangeable_frank_09.png" width="400" height="365">
</div>



### References

[1] Embrechts P., Lindskog F., McNeil A., *Modeling Dependence with Copulas and Applications to Risk Management* September 10, 2001.

[2] Hofert M., *Sampling Archimedean copulas* Fakultat fur Mathematik und Wirtschaftswissenschaften UNIVERSITAT ULM Preprint Series, Version of 2008-05-16. 

[3] Hofert M., Machler M., *Nested Archimedean Copulas Meet R— The nacopula Package*.
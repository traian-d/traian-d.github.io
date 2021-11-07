---
layout: post
title:  "Dependence and the Curse of Dimensionality"
date:   2020-05-20 22:47:00 +0100
background: '/img/bg-post.jpg'
---

[The curse of dimensionality](https://en.wikipedia.org/wiki/Curse_of_dimensionality) is a generic name for a host of problems that can arise when working in high dimensional spaces. It seems to me that the crux of the problem here is the way metrics behave with an increase in the number of dimensions. One nice illustration of the concept can be found in section 2.5 of [1]. There the authors consider the case of running a nearest-neighbor algorithm on a set of points that are uniformly distributed in a large number of dimensions. They notice that the size of a "local" neighborhood as a share of the total dataset grows quite quickly with the number of dimensions. This means essentially that it becomes harder and harder to have distinct neighborhoods for each point - once the number of dimensions is large enough, most points will be in the "local" neighborhood of most other points. 

A second related problem is that of sparsity: while a uniform sample of size 100 will pretty accurately cover a unit interval, it will be totally inadequate in, let's say, a 10-dimensional unit hypercube. As the dimensionality grows, you will need exponentially more points to adequately cover the volume - in practice this quickly leads to unfeasibly large sample size requirements.
 
Finally, yet another related way of looking at the problem is expressed in [2], which considers the behavior of distance metrics in a high dimensional space. For two d-dimensional vectors $(x_1, ..., x_d), (y_1, ..., y_d)$, a metric $L_k = \sum_{i=1}^d{[(x_i - y_i)^k]^{1/k}}$ and $\|\|x\|\|_k$ denoting the k-norm of a vector x, the authors denote the largest/smallest k-norm of a set of points as $Dmax_d^k$ and $Dmax_d^k$ respectively. They then define the *relative contrast* as:
$
\frac{Dmax_d^k - Dmin_d^k}{Dmin_d^k}
$.

Using these definitions, *Theorem 1* of [2] states that:

$
\lim_{d \to \infty} var\left(\frac{\|\|X_d\|\|_k}{\mathbb{E}[\|\|X_d\|\|_k]}\right) = 0 \Rightarrow \frac{Dmax_d^k - Dmin_d^k}{Dmin_d^k} \overset{p}{\to} 0
$

This means that as the number of dimensions grows, it becomes harder and harder to distinguish between the nearest and furthest points - distances between any pair of points simply end up being more and more similar to each other. Moreover, this also means that the relative contrast can itself be used to measure how meaningful a certain distance metric is for a given dataset and order *k*. 

Investigating this meaningfulness for a range of k's, the authors conclude that: *the Manhattan distance metric ($L_1$ norm) is consistently more preferable than the Euclidean distance metric ($L_2$ norm) for high dimensional data mining applications.*; introducing the concept of a *fractional* distance metric ($k \in (0, 1)$), they also add that *fractional distance metrics provide more meaningful results both from the theoretical and empirical perspective*.

## Correlated RV's

With these observations in mind, I thought it would be interesting to see how things play out when the data is drawn not from independent, but from correlated rv's.

In [1] the authors gauge the impact of the curse of dimensionality by comparing the ratio of volumes between two hypercubes: one with a side of $l = 1$, one with a side $l'< 1$. As the number of dimensions increases, they observe that the volume ratio decreases quite rapidly. This means that in order to capture the same ratio of the volume of the big hypercube, $l'$ has to keep growing with the number of dimensions. A cube with $l' = 0.9$ will have $0.9^3 = 0.729$ of the volume of the unit cube. A 4-d hypercube on the other hand will have a share of only $0.9^4=0.656$ of the unit hypercube. 

As a benchmark, I have first replicated *figure 2.6* on page 23 of [1]. You can see that, as the dimensionality increases, you need a larger and larger side length for a given volume of the hypercube. In 10 dimensions for example, a hypercube with side length $l' < 0.6$ basically has a volume of 0:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/cod/volume_ratio_by_ndim.png" width="400" height="365">
</div>

We can now do the same for correlated data drawn from a multidimensional Frank copula (see [here](https://traian-d.github.io/2020/04/17/Sampling-Correlated-RVs.html) for details). In each case I simulated 10000 draws from copulas with various numbers of dimensions and different parameters $\theta$ (however, for each draw the $\theta$ was the same between all pairs of 1-dimensional marginals). For each $\theta$ and a range of numbers of dimensions, you can count the share of observations that lie in a hypercube with a certain side length $l'\in(0,1)$:

<table>
  <tr>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/volume_ratio_by_ndim_frank_theta_2.0.png" width="400" height="365">
      </div>
    </td>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/volume_ratio_by_ndim_frank_theta_6.0.png" width="400" height="365">
      </div>
    </td>
  </tr>
  <tr>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/volume_ratio_by_ndim_frank_theta_12.0.png" width="400" height="365">
      </div>
    </td>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/volume_ratio_by_ndim_frank_theta_20.0.png" width="400" height="365">
      </div>
    </td>
  </tr>
</table>

Setting aside the fact that the lines are a bit truncated, these plots can be compared to the one made for uncorrelated data. It can be seen quite clearly that, as the $\theta$ parameter increases (hence there's more correlation between the marginals), the size of the volume needed to capture a certain share of the observations converges to a line that doesn't differ too much w.r.t. the number of dimensions. This means that as the data becomes more correlated, the curse of dimensionality becomes less and less of a problem. 

### Contrasts

A similar approach can be used to investigate how distance metrics evolve with changes in dimensionality and correlation. To do this I have computed the relative contrast for the datasets previously simulated, for norms with orders between 0.25 and 4. What is interesting to observe in this case is how for higher $\theta$ values the contrasts don't go as quickly to 0 with an increase in dimensionality. It should however also be noticed that the magnitude of the relative contrasts still decreases significantly and becomes quite low regardless of the metric order or the correlation strength.

<table>
  <tr>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_0.25.png" width="400" height="365">
      </div>
    </td>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_4.png" width="400" height="365">
      </div>
    </td>
  </tr>
  <tr>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_8.png" width="400" height="365">
      </div>
    </td>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_10.png" width="400" height="365">
      </div>
    </td>
  </tr>
    <tr>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_16.png" width="400" height="365">
      </div>
    </td>
    <td>
      <div style="text-align:center;margin:20px 0px">
      <img src="/assets/cod/contrast_by_ndim_frank_theta_20.png" width="400" height="365">
      </div>
    </td>
  </tr>
</table>

A (not super tidy) notebook with the relevant work is available [here](https://github.com/traian-d/curse_of_dimensionality).

### References

[1] Hastie T., Tibshirani R., Friedman J., *The Elements of Statistical Learning, Second Edition*.

[2] Aggarwal C.C, Hinneburg A., Keim D.A. *On the Surprising Behavior of Distance Metrics in High Dimensional Space* (2001).

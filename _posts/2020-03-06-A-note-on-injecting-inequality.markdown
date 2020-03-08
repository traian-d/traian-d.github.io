---
layout: post
title:  "A Note on Injecting Inequality"
date:   2020-03-06 19:47:00 +0200
---


After reading Thomas Piketty's *Capital in the Twenty-First Century* I started thinking about mechanisms to inject inequality into a system.
In an abstract sense, I'm wondering how is it possible to start with human beings whose attributes (seems to me) are normally distributed and end up with great inequality of outcomes?

At first sight the question may seem simplistic since people vary on a wide range of dimensions and there are very complex social dynamics playing an important role in generating inequality (as Piketty's book shows). I decided therefore to limit myself to the following toy problem: How can you take a normally distributed random variable and produce a r.v. that has long tails (maybe even asymmetrical, with high right skewness)?

The answer is quite simple: all you need is to find a suitable transformation of the random variable. If we take the general case of a r.v. X with p.d.f. $$f_{X}$$ and a monotonically increasing function $$g$$, then $$Y = g(X)$$ will have:

$$
\begin{align*}
F_{Y}(y) &= P(Y \leq y)\\
		 &= P(g(X) \leq y)\\
		 &= P(X \leq g^{-1}(y))\\
		 &= F_{X}(g^{-1}(y))\\
and\\
f_{Y}(y) &= \frac{d}{dy}F_{X}(g^{-1}(y))\\
	  	 &=  \frac{d}{dy}(g^{-1}(y))f_{X}(g^{-1}(y))\\
\end{align*}
$$

For $$X \sim N(\mu, \sigma^{2})$$ and $$g(X) = a + bX$$ this will yield a transformed r.v. $$Y \sim N(a + b\mu, b^{2}\sigma^{2})$$. For $$g(X)=e^{X}$$ this will simply lead to a Log-normal distribution which has a longer right tail than the original $$X$$.

We can quickly verify this by drawing 10000 samples from $$X \sim N(0, 1)$$ and applying the respective transformations.

<table>
	<th>$$Y = a + bX$$</th>
	<th>$$Y = e^{X}$$</th>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/linear_transform_hist.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/exp_transform_hist.png">
			</div>
		</td>
	</tr>
</table>

As can be seen, the simulated transformations match theoretical expectations. What's most interesting is to note that the Log-normal distribution already has a lot more of a positive skew compared to either the original X, or to the linear transformation.

### A small simulation study

From these simple theoretical insights I moved on to trying to simulate how the choice of transformation functions influences data distribution after repeated applications. The setup I considered is as follows:
 * $$n$$ competitors play a game where success is defined by a single metric $$X$$.
 * In the beginning each player is assigned a *seed* value $$s$$ drawn from $$X \sim N(\mu, \sigma^{2})$$.
 * In a given round of the game, player $$p$$ will draw a value $$x_{p}$$ (time in seconds) from $$X_{p} \sim N(s_{p}, \sigma^{2}_{p})$$
 * Based on the values drawn, each player is assigned a reward according to some function (e.g. linear, exponential, player rank). This is continuously added to the stock of previously accumulated rewards.
 * After T rounds the reward distribution is analyzed.

#### Data

To make the simulation more grounded in reality, I have derived $$\mu$$, $$\sigma$$ and $$\sigma_{p}$$ from the times recorded by male athletes running the 100m dash in 2019. The data was scraped from [worldathletics.org](worldathletics.org) with a script that is available on github.

The dataset has 21455 observations corresponding to 6062 individual athletes. It seems that only times up to 11s were kept, giving the empirical distribution a truncated aspect:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/inequality/athlete_data_trunc.png">
</div>

Since I wanted to avoid this artifact, I only kept results from athletes whose mean times were <= 10.685 s. This yields a symmetric, normal-looking distribution:

<div style="text-align:center;margin:20px 0px">
<img src="/assets/inequality/athlete_data_detrunc.png">
</div>

The simulation was parametrized with empirical estimates from this de-truncated data.

#### Simulation Parameters

 * $$\mu = 10.5349$$ seconds 
 * $$\sigma = 0.1286$$ seconds 
 * $$\sigma_{p} = 0.1129$$ seconds - constant for all players, approximated roughly, as the mean of the s.d. of each athlete's times.
 * $$n = 1024$$ players
 * $$T = 1001$$ rounds

#### Reward functions

In each round, for a player $$p$$ the reward functions considered are:

 * proportional linear: $$reward_{p} = 12 - x_{p}$$
 * proportional exponential: $$reward_{p} = e^{- x_{p}}$$
 * linear rank: $$reward_{p} = n - rank_{p} + 1$$
 * Scaled Snooker rank.

 The final reward function is based on the prize money given in the World Snooker Championship in 2019. I picked this reward scheme because it was more finely graded than the one used in the athletics championships which seem to only reward the first three athletes. 

 For the sake of fairness the number of Snooker rewards was scaled to match $$n$$, the number of players (see the *snooker_rewards* function in *functions.py* for details). 

For each of the reward functions I plotted the reward distribution every 200 iterations of the simulation, as well as a boxplot of the final rewards.

#### Proportional Linear Rewards

For proportional linear rewards at the end of the simulation the totals are distributed with a higher variance than the original seed values (as expected), but there seems to be little skewness in the empirical distribution.

<table>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/simple_proportional_rewards.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/lpr_boxplot.png">
			</div>
		</td>
	</tr>
</table>

#### Proportional Exponential Rewards

In this case we see a comparatively wider distribution of final rewards, as well as a somewhat longer right tail (as expected given the Log-normal nature of each individual's reward distribution). Since the variables simulated are not i.i.d. (each player has its own mean), it's not surprising that the final distribution is more simetrical than a Log-normal.

<table>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/exp_proportional_rewards.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/epr_boxplot.png">
			</div>
		</td>
	</tr>
</table>

#### Linear Rank Rewards

It's interesting to note that rank rewards lead to a far higher variance in outcomes. This makes sense, since even though there's some randomness in the simulation, people will tend to remain in the same region of the rankings round after round.

<table>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/linear_rewards.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/lr_boxplot.png">
			</div>
		</td>
	</tr>
</table>

#### Snooker Rank Rewards

Snooker rewards are the only ones that are truly capable of producing a long-tailed distribution. After 1000 rounds we see that the median of the distribution hasn't moved very much, however the boxplot shows a lot more extreme outliers compared to all the previous reward functions.

<table>
	<tr>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/snooker_rewards.png">
			</div>
		</td>
		<td>
			<div style="text-align:center;margin:20px 0px">
			<img src="/assets/inequality/sr_boxplot.png">
			</div>
		</td>
	</tr>
</table>

### Conclusions

The main takeaway of this little simulation study is that the reward function makes all the difference. Starting from normally distributed initial abilities, a whole range of outcomes can be achieved depending of what the reward designer intends to accomplish.

The setup that best reflects the initial distribution of seed values is that of rewards proportional to a competitor's time. Whether it's a linear or an exponential reward, the final outcome distribution remains symmetrical, with relatively short tails.

Rank based rewards however lead to profoundly distorted outcome distributions. Linear rewards give a very high variance, while Snooker rewards lead to extremely long tails and hence a really lopsided distribution of outcomes.

It is important to note that in real life pretty much all sports that I can think of have rank-based rewards that lead to the great inequality shown previously. This however doesn't have to be the case - it is quite likely that, as is the case with sprinters, in most sports aptitude is normally distributed. It's the choice of rewards that actually injects the great inequality found in final outcomes.

As always, the code and data can be found on [github](https://github.com/traian-d/normal_skew). A Jupyter notebook is also there in case anybody wants to play around with the simulation.
---
title: "Stochastic volatility in the Baumeister–Hamilton SVAR"
excerpt: "Adding a single common stochastic-volatility factor to the Baumeister–Hamilton (2015) structural VAR keeps its conjugate structure, given the volatility path. It brings no new identification, but it more than halves the uncertainty about the labor-supply elasticity and makes the size of shocks depend on the state of the cycle."
date: 2026-10-03
mathjax: true
tags: [Bayesian SVAR, stochastic volatility, sign restrictions, labor market]
---

Baumeister and Hamilton (2015), BH15 from here on, show that in a set-identified structural VAR the prior keeps shaping the posterior within the identified set even in large samples. They therefore propose to state the prior explicitly, placing it directly on the contemporaneous coefficients, which can be read as the model elasticities.

The BH15 baseline is homoskedastic, yet time-varying volatility is one of the most robust features of macroeconomic data, especially in commodity markets. In this note I add the most parsimonious departure from homoskedasticity: a single common log-volatility factor that scales all structural shocks at once.

<div class="notice--info" markdown="1">
**In short**

- Conditional on the volatility path, the conjugacy at the heart of BH15 survives unchanged, so the sampler only gains two blocks.
- A single common factor brings no new identifying information: identification still rests entirely on the prior.
- It still pays off. In the US labor market it more than halves the uncertainty about the labor-supply elasticity and time-varying volatility means that a one-standard-deviation shock in 2009 moves employment 1.8 times as much as one in 1995.
</div>

## The model

The structural model is that of BH15 with a common volatility factor:

$$
A\, y_t = B\, x_{t-1} + u_t, \qquad u_t \mid \mathcal{F}_{t-1} \sim N\!\left(0,\; e^{h_t} D\right),
$$

$$
h_t = \phi\, h_{t-1} + \sigma\, \eta_t, \qquad \eta_t \sim N(0, 1), \qquad \lvert \phi \rvert < 1.
$$

where $$D = \operatorname{diag}(d_{11}, \dots, d_{nn})$$ collects the structural variances. Every one of them is scaled by the same factor, $$d_{it} = d_{ii}\, e^{h_t}$$, so the reduced-form covariance matrix moves in block: $$\Omega_t = e^{h_t}\, \Omega$$. The priors are those of BH15: a sign-restricted Student-t prior on the elements of $$A$$ and a natural-conjugate Normal–inverse-gamma prior on each equation's coefficients and variance, plus standard priors on $$\phi$$ and $$\sigma^2$$.

## Conjugacy given the volatility path

The key step in BH15 is that, once $$A$$ is fixed, each structural equation is just a Gaussian linear regression,

$$
z_i = X b_i + u_i, \qquad z_i = Y a_i, \qquad u_i \sim N(0,\, d_{ii} I_T),
$$

where $$a_i'$$ and $$b_i'$$ are the $$i$$-th rows of $$A$$ and $$B$$. With the prior $$b_i \mid d_{ii} \sim N(m_i,\, d_{ii} M_i)$$, in which the prior variance of the coefficients scales with the structural variance, both $$b_i$$ and $$d_{ii}$$ can be integrated out analytically. What is left is a closed-form marginal posterior for $$A$$:

$$
p(A \mid Y) \;\propto\; p(A)\, \lvert \det A \rvert^{T} \prod_{i=1}^{n} p(z_i \mid A).
$$

With stochastic volatility, condition on the whole path $$H = \{h_t\}$$ and divide each period by $$e^{h_t/2}$$:

$$
z^{\star}_{it} = e^{-h_t/2}\, a_i' y_t, \qquad x^{\star}_{t} = e^{-h_t/2}\, x_{t-1}.
$$

In these GLS-standardized variables every equation is again homoskedastic with variance $$d_{ii}$$, so the Normal–inverse-gamma algebra goes through line by line. The standardization Jacobian, $$\exp\big(-\tfrac{1}{2}\sum_t h_t\big)$$, depends only on the volatility path. Given $$H$$ it is a constant, so it drops out of the conditional posteriors and of the Metropolis acceptance ratio for $$A$$. The structural Jacobian $$\lvert \det A \rvert^{T}$$ is unchanged.

The common factor does not help identification, though. With a single factor the relative variances never change, $$d_{it}/d_{jt} = d_{ii}/d_{jj}$$ in every period, whereas identification through heteroskedasticity (Rigobon, 2003; Lanne, Lütkepohl and Maciejowska, 2010; Lewis, 2021) requires them to move. Identification therefore still rests entirely on the prior $$p(A)$$, as in BH15.

The Gibbs sampler has four blocks:

1. $$A \mid Y, H$$: random-walk Metropolis on the marginal posterior above, computed with the standardized data.
2. $$(b_i, d_{ii}) \mid A, H, Y$$: direct Normal–inverse-gamma draws, equation by equation.
3. $$H \mid A, B, D, Y$$: the volatility path, using the Kim, Shephard and Chib (1998) mixture approximation and forward-filtering backward-sampling.
4. $$(\phi, \sigma^2) \mid H$$: conjugate draws.

Setting $$\sigma^2 \to 0$$ freezes $$h_t = 0$$ and recovers the homoskedastic BH15 sampler. Two practical details matter. The levels of $$h_t$$ and $$d_{ii}$$ are not separately identified (only $$d_{ii}\, e^{h_t}$$ is), so I recentre $$h_t$$ to mean zero at every iteration. Otherwise it drifts as $$\phi$$ approaches one. And the sign restrictions are imposed by rejecting Metropolis proposals that violate them, which amounts to truncating $$p(A)$$. Since the unconstrained mode violates them, the chain starts at the centre of the prior.

## Application: the US labor market

I use the BH15 bivariate labor-market model: quarterly growth of real hourly compensation and of employment, eight lags, 1970:Q1–2014:Q2. Labor demand and supply shocks are separated by the BH15 prior, with a downward-sloping demand curve ($$\beta < 0$$) and an upward-sloping supply curve ($$\alpha > 0$$), plus a long-run restriction. The sampler runs 24,000 iterations and keeps the last 12,000.

**The volatility factor.** The common factor recovers the main episodes of US macroeconomic volatility: the Great Inflation, the Great Moderation and the Great Recession (Figure 1).

<figure>
  <a href="/assets/blog/bh15-sv/volatility-factor.png"><img src="/assets/blog/bh15-sv/volatility-factor.png" alt="Posterior median and 68% band of the common volatility factor, 1970 to 2014"></a>
  <figcaption><strong>Figure 1. Common volatility factor</strong> \(e^{h_t/2}\), median and 68% band. Peaks in 1975 and 1980, below one after 1984 with a trough around 0.73 in the mid-1990s, and a peak around 1.37 in 2008–09. Posterior means: \(\phi \approx 0.91\), \(\sigma^2 \approx 0.04\).</figcaption>
</figure>

**Precision.** The point estimates of the elasticities barely move, as expected when identification is unchanged, but the supply elasticity is estimated far more precisely: its posterior standard deviation falls from 0.26 to 0.11, and the upper end of its 95% interval drops from 1.12 to 0.48. With stochastic volatility, the variance of the turbulent episodes of 1973–75 and 2008–09 is attributed to $$h_t$$ rather than to a large supply elasticity, and the right tail that the homoskedastic model could not rule out collapses (Figure 2).

| | Demand slope $$\beta$$, no SV | Demand slope $$\beta$$, SV | Supply slope $$\alpha$$, no SV | Supply slope $$\alpha$$, SV |
|---|---|---|---|---|
| Median | −1.03 | −1.07 | 0.16 | 0.14 |
| Std. deviation | 0.62 | 0.67 | 0.26 | 0.11 |
| 95% interval | [−2.51, −0.11] | [−2.86, −0.25] | [0.06, 1.12] | [0.04, 0.48] |

<figure>
  <a href="/assets/blog/bh15-sv/elasticity-posteriors.png"><img src="/assets/blog/bh15-sv/elasticity-posteriors.png" alt="Posterior histograms of the demand and supply slopes with and without stochastic volatility"></a>
  <figcaption><strong>Figure 2. Elasticity posteriors</strong> with (blue) and without (grey) stochastic volatility. The demand slope \(\beta\) (left) is essentially the same. The supply slope \(\alpha\) (right) keeps its centre, but stochastic volatility removes the right tail, between 0.5 and 1.4, that the homoskedastic model leaves open.</figcaption>
</figure>

The variance-decomposition shares do not move, with wage variance about 88% demand-driven and employment variance about 85% supply-driven in both models, but their 68% bands are about 27% narrower with stochastic volatility.

**Historical decomposition.** Figure 3 compares the contributions of demand and supply shocks to employment in the two models. They share the same broad shape: supply shocks drive the build-up of employment through the 1980s and 1990s and its fall after 2008. With stochastic volatility, however, both contributions are somewhat lower, especially from the late 1990s on, and the post-2008 fall in the supply contribution is much sharper, to about −7 by 2010 against about −1 without stochastic volatility. On average, the stochastic-volatility bands are about 13% narrower.

<figure>
  <a href="/assets/blog/bh15-sv/historical-decomposition.png"><img src="/assets/blog/bh15-sv/historical-decomposition.png" alt="Historical decomposition of employment into demand and supply contributions, with and without stochastic volatility"></a>
  <figcaption><strong>Figure 3. Historical decomposition of employment</strong>: contributions of demand shocks (left) and supply shocks (right), with and without stochastic volatility. Shaded areas are 68% bands.</figcaption>
</figure>

**Unit responses.** Responses to a unit structural shock line up on impact, where they depend only on $$A$$ (Figure 4). At longer horizons they also depend on the lag coefficients, which stochastic volatility estimates by GLS, so the two models drift apart somewhat. The shapes and signs are the same and the bands overlap.

<figure>
  <a href="/assets/blog/bh15-sv/unit-irfs.png"><img src="/assets/blog/bh15-sv/unit-irfs.png" alt="Responses of wages and employment to unit demand and supply shocks, with and without stochastic volatility"></a>
  <figcaption><strong>Figure 4. Responses to a unit structural shock</strong>, cumulated to levels, with and without stochastic volatility. Shaded areas are 68% bands. Impact responses coincide, and differences at longer horizons come from the lag coefficients.</figcaption>
</figure>

**State dependence.** A one-standard-deviation shock has size $$\sqrt{d_{ii}}\, e^{h_t/2}$$, so its impulse response is the unit response scaled by the volatility of the moment. A shock in 2009 moves employment about 1.8 times as much as one in 1995, the ratio of the median volatility factor in the two years, 1.35 against 0.75 (Figure 5). The homoskedastic model has a single one-standard-deviation response. Stochastic volatility replaces it with a family that breathes with the cycle.

<figure>
  <a href="/assets/blog/bh15-sv/one-sd-employment.png"><img src="/assets/blog/bh15-sv/one-sd-employment.png" alt="Employment responses to one-standard-deviation demand and supply shocks in 2009 and in 1995"></a>
  <figcaption><strong>Figure 5. One-standard-deviation responses of employment</strong> to a demand shock (left) and a supply shock (right), dated 2009 (red, high volatility) and 1995 (blue, low volatility). Same shape, but the 2009 response is about 1.8 times larger.</figcaption>
</figure>

**Where the volatility lives.** Under homoskedasticity, the squared demand shocks are autocorrelated, the hallmark of volatility clustering: the first-order autocorrelation is 0.26 and the Ljung–Box p-value 0.001 (Figure 6). After standardizing by the volatility factor, the autocorrelation falls to 0.14 and the p-value rises to 0.14. The supply shock shows no clustering in either model. The time-varying volatility sits in the demand shock, which is also the dominant driver of wages.

<figure>
  <a href="/assets/blog/bh15-sv/acf-squared-shocks.png"><img src="/assets/blog/bh15-sv/acf-squared-shocks.png" alt="Autocorrelation and partial autocorrelation of the squared demand and supply shocks, with and without stochastic volatility"></a>
  <figcaption><strong>Figure 6. Autocorrelation (left) and partial autocorrelation (right) of the squared structural shocks</strong>, for the demand shock (top) and the supply shock (bottom), with and without stochastic volatility. Dashed lines mark the \(\pm 1.96/\sqrt{T}\) bounds, and the panel titles report Ljung–Box p-values.</figcaption>
</figure>

## Takeaways

A single common volatility factor is a cheap, clean addition to BH15. Given the volatility path the conjugate sampler is unchanged, and one extra block delivers the volatility. It does not relax the identification problem, but it pays off twice: in efficiency, by attributing crisis-era variance to $$h_t$$ rather than to the structural parameters, and in state dependence, since the size of shocks now moves with the cycle.

It also leaves something on the table. The clustering lives in the demand shock alone, so a shock-specific volatility $$h_{it}$$ would let relative variances move over time. That would deliver identification through heteroskedasticity on top of the BH15 prior and let the variance shares change over time, which is the natural next step.

## References

- Baumeister, C. and J. D. Hamilton (2015). Sign restrictions, structural vector autoregressions, and useful prior information. *Econometrica*, 83(5), 1963–1999.
- Kim, S., N. Shephard and S. Chib (1998). Stochastic volatility: likelihood inference and comparison with ARCH models. *Review of Economic Studies*, 65(3), 361–393.
- Lanne, M., H. Lütkepohl and K. Maciejowska (2010). Structural vector autoregressions with Markov switching. *Journal of Economic Dynamics and Control*, 34(2), 121–131.
- Lewis, D. J. (2021). Identifying shocks via time-varying volatility. *Review of Economic Studies*, 88(6), 3086–3124.
- Rigobon, R. (2003). Identification through heteroskedasticity. *Review of Economics and Statistics*, 85(4), 777–792.

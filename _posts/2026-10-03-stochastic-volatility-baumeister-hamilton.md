---
title: "Stochastic volatility in the Baumeister–Hamilton SVAR"
excerpt: "Adding a single common stochastic-volatility factor to the Baumeister–Hamilton (2015) structural VAR keeps its conjugate structure, given the volatility path. It brings no new identification and, on current US labor-market data, leaves the posterior medians close to the homoskedastic ones, but it measures the volatility path and makes the size of shocks depend on the state of the economy."
date: 2026-10-03
written: 2026-06-05
mathjax: true
---

Baumeister and Hamilton (2015), BH15 from here on, show that in a set-identified structural VAR the prior keeps shaping the posterior within the identified set even in large samples. They therefore propose to state the prior explicitly, placing it directly on the contemporaneous coefficients, which can be read as the model elasticities.

The BH15 baseline is homoskedastic, yet time-varying volatility is one of the most robust features of macroeconomic data, especially in commodity markets. In this note I add the most parsimonious departure from homoskedasticity: a single common log-volatility factor that scales all structural shocks at once.

<div class="notice--info" markdown="1">
**In short**

- Conditional on the volatility path, the conjugacy at the heart of BH15 survives unchanged.
- A single common factor brings no new identifying information: identification still rests entirely on the set-identified model.
- On US labor-market data for 1970–2019 it leaves the posterior medians close to the homoskedastic ones, with a somewhat tighter posterior for the demand slope and wider bands for the wage responses. What it adds is the volatility path itself: a one-standard-deviation shock in 2009 moves employment about three times as much as one in 1995.
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

1. $$A \mid Y, H$$: random-walk Metropolis on the marginal posterior above, computed with the standardized data, with the random walk on $$\log(-\beta)$$ and $$\log \alpha$$.
2. $$(b_i, d_{ii}) \mid A, H, Y$$: direct Normal–inverse-gamma draws, equation by equation.
3. $$H \mid A, B, D, Y$$: the volatility path, using the Kim, Shephard and Chib (1998) mixture approximation to $$\log \chi^2_1$$, which matches its mean $$E[\log \chi^2_1] \approx -1.27$$, and forward-filtering backward-sampling.
4. $$(\phi, \sigma^2) \mid H$$: conjugate draws.

Setting $$\sigma^2 \to 0$$ freezes $$h_t = 0$$ and recovers the homoskedastic BH15 sampler. Two practical details matter. The levels of $$h_t$$ and $$d_{ii}$$ are not separately identified (only $$d_{ii}\, e^{h_t}$$ is), so I recentre $$h_t$$ to mean zero at every iteration. Otherwise it drifts as $$\phi$$ approaches one. And the random walk for $$A$$ runs on $$\log(-\beta)$$ and $$\log \alpha$$, with the corresponding Jacobian, and its covariance is adapted during the burn-in (Haario, Saksman and Tamminen, 2001). Every proposal then satisfies the sign restrictions, and the chain explores the long right tail of $$\alpha$$, which lies against the restriction at zero and which a random walk in levels visits only rarely.

## Application: the US labor market

I use the BH15 bivariate labor-market model: quarterly growth of real hourly compensation in the nonfarm business sector and of total nonfarm payroll employment, eight lags. Labor demand and supply shocks are separated by the BH15 prior, with a downward-sloping demand curve ($$\beta < 0$$) and an upward-sloping supply curve ($$\alpha > 0$$), plus a long-run restriction. The data are the current FRED vintage (series COMPRNFB and PAYEMS) and the sample is 1970:Q1–2019:Q4. It stops before 2020 because the COVID quarters are outliers of more than fifteen standard deviations, which a Gaussian volatility process can only absorb by inflating $$\sigma^2$$ (Carriero, Clark, Marcellino and Mertens, 2024). Each model is estimated with four chains of 110,000 iterations from dispersed starting points, discarding the first 10,000. The split $$\hat{R}$$ is 1.00 for both slopes and the effective sample size of $$\alpha$$ exceeds 20,000.

**The volatility factor.** The common factor recovers the main episodes of US macroeconomic volatility: the Great Inflation, the Great Moderation and the Great Recession (Figure 1).

<figure>
  <a href="/assets/blog/bh15-sv/volatility-factor.png?v=4"><img src="/assets/blog/bh15-sv/volatility-factor.png?v=4" alt="Posterior median, 68% and 90% bands of the common volatility factor, 1970 to 2019"></a>
  <figcaption><strong>Figure 1. Common volatility factor</strong> \(e^{h_t/2}\), posterior median with 68% and 90% bands. Peaks of 1.76 in 1974:Q4 and 1.68 in 1980:Q2, mostly below one from 1984 with a trough of 0.57 in 1996, a peak of 1.87 in 2008:Q4 and a second trough of 0.60 in 2017. Posterior means: \(\phi \approx 0.92\), \(\sigma^2 \approx 0.08\).</figcaption>
</figure>

**Elasticities.** The posteriors of both slopes are close in the two models (Figure 2). The supply slope is concentrated around 0.1, well below the centre of its prior, with a long right tail: the upper end of the 95% interval is 0.63 with stochastic volatility and 0.61 without. The posterior of the demand slope overlaps heavily with its prior, as in BH15. With stochastic volatility it is somewhat tighter and slightly closer to zero, and its 95% interval is about 12% narrower.

| | Demand slope $$\beta$$, no SV | Demand slope $$\beta$$, SV | Supply slope $$\alpha$$, no SV | Supply slope $$\alpha$$, SV |
|---|---|---|---|---|
| Median | −1.12 | −0.99 | 0.13 | 0.10 |
| Std. deviation | 0.94 | 0.80 | 0.19 | 0.21 |
| 95% interval | [−3.48, −0.22] | [−3.02, −0.16] | [0.03, 0.61] | [0.02, 0.63] |

<figure>
  <a href="/assets/blog/bh15-sv/elasticity-posteriors.png?v=4"><img src="/assets/blog/bh15-sv/elasticity-posteriors.png?v=4" alt="Posterior histograms of the demand and supply slopes with and without stochastic volatility, and their prior"></a>
  <figcaption><strong>Figure 2. Posterior histograms of the elasticities</strong>, with and without stochastic volatility, and the prior density, the sign-truncated Student-t of BH15. Each histogram uses 400,000 draws.</figcaption>
</figure>

The variance-decomposition shares at a four-year horizon barely move. Wage variance is 89% demand-driven with stochastic volatility and 88% without, and employment variance is 87% supply-driven with stochastic volatility and 84% without. The 68% bands are about as wide in the two models.

**Historical decomposition.** Figures 3 and 4 show the contribution of each shock to the eight-quarter growth of employment and wages in the model with stochastic volatility. Summing the contributions over an eight-quarter window, rather than cumulating them from 1970, keeps their uncertainty from accumulating over the sample. Supply shocks account for the falls in employment growth around every recession, with troughs of about −2.8 percentage points in 1975, −4.8 in 1983, −4.3 in 1992, −4.7 in 2002 and −8.4 at the end of 2009. Demand shocks move employment growth by less than two points. The homoskedastic medians, not shown, differ from these by about one point at most, because the realized structural shocks are the same residuals in both models and the posteriors of $$A$$ and $$B$$ are close.

<figure>
  <a href="/assets/blog/bh15-sv/historical-decomposition.png?v=4"><img src="/assets/blog/bh15-sv/historical-decomposition.png?v=4" alt="Contributions of demand and supply shocks to eight-quarter employment growth, model with stochastic volatility"></a>
  <figcaption><strong>Figure 3. Historical decomposition of employment</strong>: contributions of demand shocks (left) and supply shocks (right) to eight-quarter employment growth, model with stochastic volatility. Posterior median with 68% and 90% bands.</figcaption>
</figure>

Wage growth is driven mostly by demand shocks, which subtract about 5.7 points in 1975 and 6.7 in 2009 and add about 5.0 in 1999 (Figure 4). Around recessions supply shocks push wages up, by about 3.4 points in 2009, the mirror image of their negative contribution to employment.

<figure>
  <a href="/assets/blog/bh15-sv/historical-decomposition-wages.png?v=4"><img src="/assets/blog/bh15-sv/historical-decomposition-wages.png?v=4" alt="Contributions of demand and supply shocks to eight-quarter wage growth, model with stochastic volatility"></a>
  <figcaption><strong>Figure 4. Historical decomposition of wages</strong>: contributions of demand shocks (left) and supply shocks (right) to eight-quarter wage growth, model with stochastic volatility. Posterior median with 68% and 90% bands.</figcaption>
</figure>

**Unit responses.** On impact, the response to a unit structural shock depends only on $$A$$, so the two models differ as much as their posteriors of $$A$$ do: the impact response of wages to a demand shock is 0.90 with stochastic volatility and 0.78 without (Figure 5). At longer horizons the responses also depend on the lag coefficients, which stochastic volatility estimates by GLS. The shapes and signs are the same and the bands overlap. For wages the stochastic-volatility bands are wider.

<figure>
  <a href="/assets/blog/bh15-sv/unit-irfs.png?v=4"><img src="/assets/blog/bh15-sv/unit-irfs.png?v=4" alt="Responses of wages and employment to unit demand and supply shocks, with and without stochastic volatility"></a>
  <figcaption><strong>Figure 5. Responses to a unit structural shock</strong>, cumulated to levels, with and without stochastic volatility. Bands are 68% intervals.</figcaption>
</figure>

**State dependence.** A one-standard-deviation shock has size $$\sqrt{d_{ii}}\, e^{h_t/2}$$, so its impulse response is the unit response scaled by the volatility of the moment. A shock in 2009 moves employment about three times as much as one in 1995, the ratio of the median volatility factor in the two years, 1.81 against 0.61 (Figure 6). The homoskedastic model has a single one-standard-deviation response.

<figure>
  <a href="/assets/blog/bh15-sv/one-sd-employment.png?v=4"><img src="/assets/blog/bh15-sv/one-sd-employment.png?v=4" alt="Employment responses to one-standard-deviation demand and supply shocks in 2009 and in 1995"></a>
  <figcaption><strong>Figure 6. One-standard-deviation responses of employment</strong> to a demand shock (left) and a supply shock (right), dated 2009 (high volatility) and 1995 (low volatility). Bands are 68% intervals.</figcaption>
</figure>

**Where the volatility lives.** Figure 7 shows the autocorrelations of the squared standardized shocks, computed draw by draw. Under homoskedasticity both squared shocks are autocorrelated at the first lag, the hallmark of volatility clustering, more strongly for demand: the posterior median of the first-order autocorrelation is 0.26 for demand and 0.22 for supply, and the Ljung–Box test rejects the null of no autocorrelation up to lag 20, with median p-values below 0.001 and about 0.01. After standardizing by the volatility factor the first-order autocorrelations fall to 0.10 and 0.12, inside the $$\pm 1.96/\sqrt{T}$$ bounds, and the median p-values rise to 0.18 and 0.30, so the null is no longer rejected at the 5% level. The spikes at lags 11 and 12 of the demand shock also fall inside the bounds. With the common factor no posterior median of the autocorrelations or partial autocorrelations lies outside the bounds at any lag up to 20.

<figure>
  <a href="/assets/blog/bh15-sv/acf-squared-shocks.png?v=4"><img src="/assets/blog/bh15-sv/acf-squared-shocks.png?v=4" alt="Autocorrelation and partial autocorrelation of the squared demand and supply shocks, with and without stochastic volatility"></a>
  <figcaption><strong>Figure 7. Autocorrelation (left) and partial autocorrelation (right) of the squared structural shocks</strong>, for the demand shock (top) and the supply shock (bottom), with and without stochastic volatility. Posterior medians and 68% intervals across draws. The grey band marks the \(\pm 1.96/\sqrt{T}\) bounds. Median p-values of the Ljung–Box test of no autocorrelation up to lag 20: demand below 0.001 without and 0.18 with stochastic volatility, supply 0.01 and 0.30.</figcaption>
</figure>

## Takeaways

A single common volatility factor is a cheap addition to BH15. Given the volatility path the conjugate sampler is unchanged, and one extra block delivers the volatility. It does not relax the identification problem, and the posterior medians of the elasticities, the variance decomposition and the historical decomposition stay close to the homoskedastic ones. What it adds is a measured volatility path and shock sizes that move with the state of the economy.

It also leaves something on the table. With a single factor the relative variances of the shocks are fixed by construction. A shock-specific volatility $$h_{it}$$ would let them move over time, which would deliver identification through heteroskedasticity on top of the BH15 prior. Extending the sample beyond 2019 requires, in addition, a treatment of the COVID outliers.

[Code](https://github.com/jorgearenasamoros/bh15-stochastic-volatility){: .btn .btn--primary}

## References

- Baumeister, C. and J. D. Hamilton (2015). Sign restrictions, structural vector autoregressions, and useful prior information. *Econometrica*, 83(5), 1963–1999.
- Carriero, A., T. E. Clark, M. Marcellino and E. Mertens (2024). Addressing COVID-19 outliers in BVARs with stochastic volatility. *Review of Economics and Statistics*, 106(5), 1403–1417.
- Haario, H., E. Saksman and J. Tamminen (2001). An adaptive Metropolis algorithm. *Bernoulli*, 7(2), 223–242.
- Kim, S., N. Shephard and S. Chib (1998). Stochastic volatility: likelihood inference and comparison with ARCH models. *Review of Economic Studies*, 65(3), 361–393.
- Lanne, M., H. Lütkepohl and K. Maciejowska (2010). Structural vector autoregressions with Markov switching. *Journal of Economic Dynamics and Control*, 34(2), 121–131.
- Lewis, D. J. (2021). Identifying shocks via time-varying volatility. *Review of Economic Studies*, 88(6), 3086–3124.
- Rigobon, R. (2003). Identification through heteroskedasticity. *Review of Economics and Statistics*, 85(4), 777–792.

# fundamentalAlphaCapture

## **Systematic Fundamental Alpha Capture**

*An Exploration into developing an alpha capture model, using principal axis factoring. The goal of our project is to determine unique alphas from multiple different strategies and construct an optimal portfolio.*

---

# 1 Introduction

<br>

## 1.1 Idea & Motivation <a id="1-1"></a>

The core idea behind this strategy is to extract alpha from sell-side analyst ratings while ensuring market neutrality through principal axis factoring (PAF). Sell-side analysts provide ratings on stocks, and their updates often contain valuable information about expected future performance. By systematically capturing signals from these ratings and filtering out market-wide risk factors, we aim to construct a portfolio with pure alpha exposure aimed at exclusively capturing the alpha coming from fundamental research.

Additionally, we incorporate insider ownership changes as an auxiliary signal. Corporate insiders typically have a deeper understanding of their company's fundamentals. If insiders are increasing their ownership, it can be considered a bullish signal, whereas a decrease in ownership can be interpreted as a bearish signal. Combining these two signals, we create an aggregate indicator to drive our stock selection and portfolio construction. This signal further augments our alpha from fundamental ratings by research analysts as those ratings are not frequently updated and are on a discrete scale from 1–5 indicating there's a lot of room for interpretation (e.g., a 4-rated stock may move to a 5, a strong-buy, or to a 3, hold, rating). We can further add more signals to the strategy to gauge the potential direction of future rating changes (“momentum” of research ratings).

The primary motivation is to develop a market-neutral strategy that maximizes alpha while minimizing exposure to systematic risk factors. This aligns with quantitative investment methodologies that seek to isolate idiosyncratic stock performance from broader market trends. Most traditional absolute return strategies aim to neutralize exposure to known factors like the Fama–French factors or those provided by MSCI Barra. Our hypothesis with this project, though, is that the factors and their associated risk premia are dynamic, and treating them as static is not ideal. We thus compute “statistical” factors via principal axis factoring, which accounts for idiosyncratic variance in each asset.

## 1.2 Conceptual Framework <a id="1-2"></a>

### 1.2.1 Stock Selection Framework

Sell-side analysts adjust their ratings based on new information, including earnings reports, industry trends, and macroeconomic changes. Insider ownership further augments this information as company insiders may adapt more quickly to changes in fundamentals.

We construct the following signal scores (all z‑scores):

* `Research Score = z_score(Bloomberg Ratings)`
* `Insider Score  = z_score(Δ Insider Ownership)`
* `Sentiment Score = z_score(Sentiment Data)`

We then combine these scores equally:

```text
Combined Score = (Research Score + Insider Score + Sentiment Score) / 3
```

Different weighting schemes can be tested, but this equal‐weight aggregate drives our long/short selection.

### 1.2.2 Principal Axis Factoring (PAF) for Market Neutrality

To ensure market neutrality, we decompose the asset return covariance matrix into common factors plus specific variances using PAF.

```text
Sigma = Lambda * Lambda' + Psi
```

* `Sigma` is the covariance (or correlation) matrix of asset returns.
* `Lambda` contains the factor loadings.
* `Psi` is the diagonal matrix of specific variances.

The key difference from PCA is that PAF replaces the diagonal of the correlation matrix with estimated communalities (the variance explained by common factors), then iterates:

1. Initialize communalities vector `u0` (e.g., initial guesses).
2. Form adjusted matrix:

   ```text
   Sigma_adj = Sigma - I + diag(u0)
   ```
3. Compute singular value decomposition:

   ```text
   Sigma_adj = Q * D * Q'  (D is diagonal of eigenvalues)
   ```
4. Obtain factor loadings:

   ```text
   F = Q * sqrt(D)
   ```
5. Update communalities `u_t` = sum of squared rows of `F`.
6. Repeat steps 2–5 until `|u_t - u_{t-1}| <= 1e-3`.

After convergence, we extract the top `k` factors (columns of `F`). We then regress each asset return series on these factor returns to obtain exposures `beta` and idiosyncratic alphas `alpha_i`:

```text
r_i = beta_i * R^k + alpha_i + epsilon_i
```

where `R^k` are the factor return time series.

For market neutrality, we solve the portfolio optimization:

```text
Minimize: portfolio volatility
Subject to:
- Net exposure to each PAF factor = 0
- Sum of weights = 0 (dollar neutral)
- |weight_i| <= max_weight_i
```

This yields a portfolio with minimal systematic risk and pure fundamental alpha exposure.

### References:

* Fama, E. F., & French, K. R. (1993). Common risk factors in the returns on stocks and bonds.
* Jegadeesh, N., & Titman, S. (1993). Returns to buying winners and selling losers: Implications for stock market efficiency.
* Asquith, P., Mikhail, M. B., & Au, A. S. (2005). Information content of equity analyst reports.

## 1.3 Methodology <a id="1-3"></a>

### 1.3.1 Data Sources

* Bloomberg: Sell-side analyst ratings and market returns
* Bloomberg: Insider ownership changes
* Google Trends: Sentiment Analysis
* Yahoo Finance: Market data

### 1.3.2 Signal Construction

1. Analyst Ratings Signal: compute rating changes and normalize.
2. Insider Ownership Signal: compute percentage changes and normalize.
3. Google Trends Signal: extract and normalize.
4. Aggregate Signal: equal-weight sum of the three signals.

### 1.3.3 Stock Selection

* Long Portfolio: top N stocks by combined signal.
* Short Portfolio: bottom N stocks by combined signal.

### 1.3.4 Portfolio Construction & Optimization

1. Compute PAF factor loadings from historical returns.
2. Optimize weights to neutralize PAF exposures and minimize volatility.
3. Apply constraints (dollar neutrality, weight caps).
4. Rebalance frequency: weekly, monthly, or quarterly.

### 1.3.5 Execution & Monitoring

* Backtest across multiple periods.
* Evaluate Sharpe ratio, alpha, and factor neutrality.
* Update parameters regularly based on new data.

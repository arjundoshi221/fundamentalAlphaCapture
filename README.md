# fundamentalAlphaCapture

## **Systematic Fundamental Alpha Capture**

*An Exploration into developing an alpha capture model, using principal axis factoring. The goal of our project is to determine unique alphas from multiple different strategies and constructs an optimal portfolio.*

---------------------------

# 1 Introduction

<br> 

## 1.1 Idea & Motivation <a id="1-1"></a>

The core idea behind this strategy is to extract alpha from sell-side analyst ratings while ensuring market neutrality through principal axis factoring (PAF). Sell-side analysts provide ratings on stocks, and their updates often contain valuable information about expected future performance. By systematically capturing signals from these ratings and filtering out market-wide risk factors, we aim to construct a portfolio with pure alpha exposure aimed at exclusively capturing the alpha coming from fundamental research.

Additionally, we incorporate insider ownership changes as an auxiliary signal. Corporate insiders typically have a deeper understanding of their company's fundamentals. If insiders are increasing their ownership, it can be considered a bullish signal, whereas a decrease in ownership can be interpreted as a bearish signal. Combining these two signals, we create an aggregate indicator to drive our stock selection and portfolio construction. This signal further augments our alpha from fundamental ratings by research analysts as those ratings are not frequently updated and are on a discrete scale from 1-5 indicating there's a lot of room for interpretation (e.g. a 4-rated stock may be a buy moving to a 5, a strong-buy, or moving to a 3, hold, rating). We can further add more signals to the strategy to further augment our research rating signal to try to gauge the 'momentum' (where might the rating go in the future) of research ratings.

The primary motivation is to develop a market-neutral strategy that maximizes alpha while minimizing exposure to systematic risk factors. This aligns with quantitative investment methodologies that seek to isolate idiosyncratic stock performance from broader market trends. Most traditional absolute return strategies aim to neutralize exposure to known factors like the Fama-French Factors or those provided by MSCI Barra. Our hypothesis with this project, though, is that the factors, and their associated risk premia, are often dynamic and, thus, treating the factors as static is not an ideal solution. We thus try to compute 'statistical' factors and try to compute 'principal components' of our universe of assets which explain most of the variance of assets in our universe. However, instead of using the traditional Principal Component Analysis approach, which assumes that the principal components _completely_ explain all the variance in the market and that there is no idiosyncratic variance attached to stocks themselves, we use an alternative approach called the 'Pricipal Axis Factoring' (PAF). PAF assumes that there is an idiosyncratic component to variance of assets and adjusts the correlation matrix accordingly by replacing the diagonal elements with something called 'communalities'. We will explain this further in the next section.

## 1.2 Conceptual Framework <a id="1-2"></a>

### 1.2.1 Stock Selection Framework
Sell-side analysts adjust their ratings based on new information, including earnings reports, industry trends, and macroeconomic changes. Insider ownership further augments this information as company insiders may be quicker to adapt to changes in company fundamentals. We first create factor scores by normalizing research ratings data and insider ownership data:

$$
Research \; Score = z_{score}(Bloomberg \; Ratings)
$$
$$
Insider \; Score = z_{score}(\Delta Insider \; Ownership)
$$

$$
Sentiment \; Score = z_{score}(Sentiment \; Score)
$$

We then create a combined score by equally weighting these two scores:
$$

Combined \; Score = {(Research \; Score + Insider \; Score + Sentiment \; Score)\over3}
$$

We have more details in 11.2 

Through the course of this analysis, we will try different weighting schemes for combining these individual signals. We will use this combined score for identifying investment opportunities and stocks we may want to go long or short on.

### 1.2.2 Principal Axis Factoring (PAF) for Market Neutrality
To ensure our portfolio is market-neutral, we extract the primary market risk factors using Principal Axis Factoring (PAF). Given a return matrix $ X $, the covariance matrix $ \Sigma $ is decomposed as:
$$
\Sigma = \Lambda \Lambda' + \Psi
$$
where $ \Lambda $ represents factor loadings, and $ \Psi $ is the specific variance matrix. We extract the first $ k $ components that explain the most variance and construct our portfolio such that exposure to these factors is minimized. The essential difference from standard PCA is the specific variance matrix, $\Psi$. We implement this approach by using 'communalities'. The diagonal of a correlation matrix is always 1, which is an asset's correlation to itself. It can be broken down as follows:
$$
communality + idiosyncratic\; volatility\; component = 1
$$

The 'communalities' ($u$) are the proportion of variance explained through the common factors in the market. Thus, for Principal Axis Factoring, we replace the diagonal of the correlation matrix with the vector of communalities $u$. We start with an initial estimate of communalities and iteratively apply Singular Value Decomposition such that each communality is the sum of squared factor loadings. We run the iterations until we hit our tolerance level. The communalities are computed as follows:
1. Start with an initial estimate $u_0$
2. Reduce the correlation matrix: $\Sigma_{adj} = \Sigma - I + diag(u_0)$
3. Apply Singular Value Decomposition to the correlation matrix: $\Sigma_{adj} = Q\Lambda Q^{-1}$
4. Compute Factor Loadings by adjusting with the factor variances: $F=Q\Lambda^{0.5}$
5. Compute new communalities, u_t, as sum of squared factor loadings (for each asset)
6. Repeat till $|u_t - u_{t-1}| <= 10^{-3}$

This process will give us $k$ principal components which we will treat as our factors. We now decompose the return of each assets as follows:
$$
r_{i} = \beta R^k + \alpha_i +  \epsilon_i
$$

Where $R^k$ is the matrix of factor returns computed using the principal components. For market neutrality, we regress our portfolio's returns and short the $\beta$ exposures to the computed principal components. We define the optimization problem as follows:
$$
\min \sigma_p
$$
$$
Subject \; To:
$$
$$
\beta = 0
$$

We restrict our buy-universe to the top performing stocks according to our buy signal and the short-universe as the bottom performing stocks. The objective of minimizing volatility of the portfolio is in line with the idea of a market neutral strategy such that we are generating a virtually 'risk-free' return capturing only the $\alpha$. For practical considerations, we'll also add a min/max weight constraint so that we don't get a trivial 0 allocation portfolio.

### References:
- Fama, E. F., & French, K. R. (1993). *Common risk factors in the returns on stocks and bonds.*
- Jegadeesh, N., & Titman, S. (1993). *Returns to buying winners and selling losers: Implications for stock market efficiency.*
- Asquith, P., Mikhail, M. B., & Au, A. S. (2005). *Information content of equity analyst reports.*

## 1.3 Methodology <a id="1-3"></a>

### 1.3.1 Data Sources
- **Bloomberg**: Sell-side analyst ratings and market returns
- **Bloomberg**: Insider ownership changes
- **Google Trends Data**: Sentiment Analysis
- **Yahoo Finance**: Google Trends Data

### 1.3.2 Signal Construction
1. **Analyst Ratings Signal**: Compute changes in analyst ratings and normalize them across stocks.
2. **Insider Ownership Signal**: Compute percentage changes in insider ownership and normalize.
3. **Google Trends Signal**: Extract Google Trends data and normalize.
4. **Aggregate Signal**: Combine the three signals using a weighted sum approach.

### 1.3.3 Stock Selection
- **Long Portfolio**: Top 40/50/100 stocks based on the aggregate signal.
- **Short Portfolio**: Bottom 40/50/100 stocks based on the aggregate signal.

### 1.3.4 Portfolio Construction & Optimization
1. Compute factor loadings from historical returns using PAF and extract the top $ k $ components.
2. Optimize the portfolio to minimize volatility while ensuring:
   - Net market exposure is zero.
   - Exposure to top $ k $ market factors is zero.
3. Implement a rebalance frequency (weekly, monthly, or quarterly).

### 1.3.5 Execution & Monitoring
- Backtest strategy performance over multiple timeframes.
- Evaluate Sharpe ratio, alpha, and factor neutrality.
- Regularly update the model parameters based on new data.

This methodology provides a structured approach to capturing alpha from analyst ratings while systematically neutralizing exposure to known market factors.

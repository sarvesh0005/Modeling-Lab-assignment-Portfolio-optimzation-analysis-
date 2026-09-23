# Monte Carlo-Based Portfolio Risk & Optimization Analysis

## 1. Executive Summary

This lab develops a compact quantitative portfolio research workflow
using eight equities and historical daily market data from 2019 through
2023. The analysis progresses from data preparation and return/risk
estimation to portfolio construction, constrained minimum-variance
optimization, efficient-frontier construction, mean-variance
optimization (MVO), out-of-sample testing, and Monte Carlo simulation.

The core research question is:

> **How do different portfolio construction rules behave when evaluated
> using historical data, and what does a Monte Carlo model imply about
> their short-horizon return and risk distributions?**

Three portfolio constructions are compared:

1.  Equal-Weight Portfolio
2.  Long-only Minimum-Variance Portfolio
3.  Long-only Mean-Variance Portfolio

The notebook deliberately separates the final portfolio construction
from the final evaluation:

$$
\text{2019--2022 estimation}
\rightarrow
\text{portfolio construction}
\rightarrow
\text{2023 out-of-sample evaluation}
$$

The final 2023 results were:

  -----------------------------------------------------------------------
  Portfolio       Annualized     Annualized   Sharpe Ratio        Maximum
                      Return     Volatility                      Drawdown
  ----------- -------------- -------------- -------------- --------------
  Equal               78.63%         19.85%          3.960        -10.06%
  Weight                                                   

  Minimum              6.03%         14.23%          0.423        -13.96%
  Variance                                                 

  MVO                118.96%         28.11%          4.231        -14.20%
  -----------------------------------------------------------------------

The Monte Carlo experiment used 10,000 multivariate normal daily return
scenarios estimated from the 2019--2022 training period:

  -----------------------------------------------------------------------
  Portfolio       Mean Daily          Daily 5th Percentile Probability of
                      Return     Volatility                          Loss
  ----------- -------------- -------------- -------------- --------------
  Equal              0.0831%        1.7678%       -2.8202%         47.98%
  Weight                                                   

  Minimum            0.0365%        1.2153%       -1.9535%         48.69%
  Variance                                                 

  MVO                0.1651%        2.4484%       -3.8474%         47.56%
  -----------------------------------------------------------------------

These results should be interpreted as an empirical study of this
specific asset universe and sample period, not as evidence that one
portfolio construction method is universally superior.

------------------------------------------------------------------------

# 2. Research Objective

The project was designed as a fundamental introduction to portfolio
optimization and Monte Carlo simulation.

The objectives were to:

-   transform historical prices into return data suitable for portfolio
    analysis;
-   estimate expected returns and covariance structure;
-   construct a simple equal-weight benchmark;
-   formulate and solve a constrained minimum-variance portfolio
    problem;
-   construct an efficient frontier;
-   formulate mean-variance optimization with different risk-aversion
    parameters;
-   evaluate portfolios on a genuinely unseen 2023 period;
-   use Monte Carlo simulation to examine the distribution of
    hypothetical portfolio returns;
-   compare historical out-of-sample results with model-based simulated
    risk characteristics.

The workflow is therefore:

$$
\text{Prices}
\rightarrow
\text{Returns}
\rightarrow
(\mu,\Sigma)
\rightarrow
\text{Portfolio Construction}
\rightarrow
\text{Optimization}
\rightarrow
\text{Out-of-Sample Testing}
\rightarrow
\text{Monte Carlo Analysis}
$$

------------------------------------------------------------------------

# 3. Data and Universe

## 3.1 Asset universe

Eight equities were used:

-   AAPL
-   MSFT
-   GOOGL
-   AMZN
-   NVDA
-   META
-   JPM
-   JNJ

The prices were downloaded using `yfinance` with adjusted prices.

The requested data window was:

$$
\text{2019-01-01} \leq t < \text{2024-01-01}
$$

The actual observations were:

-   Start: **2019-01-02**
-   End: **2023-12-29**
-   Price observations: **1,258**
-   Assets: **8**
-   Daily return observations: **1,257**

No missing values or duplicate dates were found after the data audit.

------------------------------------------------------------------------

# 4. Why Prices Were Normalized for Visualization

Raw stock prices are not directly comparable because different assets
start at very different price levels.

For visualization, every asset was rebased to 100 on the first
observation:

$$
P_{t,\text{normalized}}
=
\frac{P_t}{P_0}\times100
$$

This answers:

> If 100 units had been invested in each asset at the beginning, how
> would the value have evolved?

Normalization was used **only for visualization**. It was not used as
the input to portfolio optimization.

The optimization pipeline instead uses returns:

$$
P_t
\rightarrow
R_t
\rightarrow
\mu,\Sigma
$$

------------------------------------------------------------------------

# 5. Return Construction

Simple daily returns were calculated as:

$$
R_t
=
\frac{P_t-P_{t-1}}{P_{t-1}}
$$

equivalently,

$$
R_t
=
\frac{P_t}{P_{t-1}}-1
$$

In the dataset, the return distributions show meaningful dispersion and
large positive/negative daily observations. For example, the observed
daily-return range included substantial movements for several assets,
particularly the higher-volatility technology names.

This motivates using both expected return and covariance structure
rather than treating every asset independently.

------------------------------------------------------------------------

# 6. Asset-Level Return and Risk Analysis

Daily mean and standard deviation were estimated:

$$
\bar{R}_i
=
\frac{1}{T}
\sum_{t=1}^{T}R_{i,t}
$$

$$
\sigma_i
=
\sqrt{
\frac{1}{T-1}
\sum_{t=1}^{T}
(R_{i,t}-\bar{R}_i)^2
}
$$

For a simple annualization convention of 252 trading days:

$$
\mu_i^{annual}
=
252\bar{R}_i
$$

and

$$
\sigma_i^{annual}
=
\sqrt{252}\sigma_i
$$

The full-sample annualized estimates were:

  Asset     Annualized Return   Annualized Volatility
  ------- ------------------- -----------------------
  AAPL                 37.80%                  32.23%
  AMZN                 19.84%                  35.22%
  GOOGL                24.60%                  31.81%
  JNJ                   8.78%                  19.86%
  JPM                  18.90%                  31.90%
  META                 28.91%                  43.63%
  MSFT                 32.03%                  30.49%
  NVDA                 67.18%                  51.77%

These are sample estimates over the selected historical period. They are
not forecasts guaranteed to hold in the future.

A notable empirical feature is that higher estimated return is
accompanied by materially different volatility across assets. NVDA, for
example, had the highest estimated annualized return and also the
highest estimated volatility in this sample.

------------------------------------------------------------------------

# 7. Correlation Analysis

The correlation matrix was calculated from daily returns:

$$
\rho_{ij}
=
\frac{
\operatorname{Cov}(R_i,R_j)
}{
\sigma_i\sigma_j
}
$$

The observed correlations were all positive in this particular
eight-asset universe, ranging approximately from **0.23 to 0.76**.

Examples:

-   MSFT--AAPL: approximately 0.76
-   MSFT--GOOGL: approximately 0.76
-   AMZN--JNJ: approximately 0.23

This matters because portfolio risk depends not only on the volatility
of individual assets but also on how their returns co-move.

A portfolio containing assets with imperfect correlation can have lower
volatility than the weighted average of individual volatilities.

------------------------------------------------------------------------

# 8. Covariance Matrix

The daily covariance matrix was estimated as:

$$
\Sigma_{ij}
=
\operatorname{Cov}(R_i,R_j)
$$

The annualized covariance matrix was obtained using:

$$
\Sigma_{annual}
=
252\Sigma_{daily}
$$

The covariance matrix is the central risk input for the optimization
problem.

For a portfolio with weights

$$
w=
\begin{bmatrix}
w_1 & w_2 & \cdots & w_N
\end{bmatrix}^T
$$

portfolio variance is:

$$
\boxed{
\sigma_p^2=w^T\Sigma w
}
$$

and portfolio volatility is:

$$
\boxed{
\sigma_p=\sqrt{w^T\Sigma w}
}
$$

The quadratic form is important because it captures both:

1.  individual asset variances; and
2.  cross-asset covariance terms.

------------------------------------------------------------------------

# 9. Equal-Weight Benchmark

Before optimization, an equal-weight portfolio was constructed as a
benchmark.

For eight assets:

$$
w_i=\frac{1}{8}=0.125
$$

Therefore:

$$
\sum_{i=1}^{8}w_i=1
$$

The portfolio expected return is:

$$
\boxed{
E[R_p]=w^T\mu
}
$$

For the full historical sample, the equal-weight portfolio produced:

-   Expected annual return: **29.76%**
-   Expected annual volatility: **26.77%**
-   Expected Sharpe ratio: **1.111**

The realized historical annualized return was approximately **29.89%**,
with annualized volatility of **26.77%** and a Sharpe ratio of
**1.117**.

Maximum drawdown was:

$$
\boxed{-39.26\%}
$$

The equal-weight portfolio serves as a useful benchmark because it
requires no optimization assumptions beyond equal allocation.

------------------------------------------------------------------------

# 10. Minimum-Variance Portfolio

## 10.1 Objective

The minimum-variance portfolio solves:

$$
\boxed{
\min_w\quad w^T\Sigma w
}
$$

subject to:

$$
\boxed{
\sum_{i=1}^{N}w_i=1
}
$$

and the long-only constraints:

$$
\boxed{
w_i\geq0
}
$$

Together:

$$
\boxed{
\begin{aligned}
\min_w \quad & w^T\Sigma w\\
\text{subject to}\quad
&\sum_{i=1}^{N}w_i=1\\
&w_i\geq0
\end{aligned}
}
$$

## 10.2 Numerical method

The notebook used `scipy.optimize.minimize` with the **SLSQP** method.

SLSQP is appropriate here because the problem contains:

-   a nonlinear quadratic objective;
-   an equality constraint;
-   bound constraints.

The optimizer started from the equal-weight portfolio.

The optimization returned successfully.

------------------------------------------------------------------------

# 11. Minimum-Variance Result

Using the full historical sample, the optimizer produced:

-   Portfolio variance: **0.034368**
-   Annualized volatility: **18.54%**
-   Expected annual return: **11.82%**
-   Expected Sharpe ratio: **0.638**

The optimized allocation was concentrated in a small number of assets.

Using the actual dataframe column order, the full-sample optimization
weights correspond to approximately:

  Asset     Minimum-Variance Weight
  ------- -------------------------
  AAPL                        0.00%
  AMZN                       13.73%
  GOOGL                       3.98%
  JNJ                        73.51%
  JPM                         8.78%
  META                        0.00%
  MSFT                        0.00%
  NVDA                        0.00%

The concentration is a useful illustration of an important property of
unconstrained-in-concentration minimum-variance optimization:

> Minimizing variance does not necessarily produce a diversified-looking
> portfolio.

The optimizer only cares about the specified mathematical objective and
constraints. If one combination of assets provides the lowest estimated
variance, the optimizer can assign a very large weight to that
combination.

The full-sample historical maximum drawdown of this portfolio was
approximately **-26.03%**.

------------------------------------------------------------------------

# 12. Efficient Frontier

The minimum-variance portfolio answers:

> What is the lowest-risk feasible portfolio?

The efficient-frontier analysis extends this by asking:

> For a specified expected return, what is the minimum achievable
> variance?

For each target return $R_{target}$, the notebook solved:

$$
\boxed{
\min_w\quad w^T\Sigma w
}
$$

subject to:

$$
\boxed{
w^T\mu=R_{target}
}
$$

and:

$$
\boxed{
\sum_iw_i=1
}
$$

and:

$$
\boxed{
w_i\geq0
}
$$

The target returns were generated over the range:

$$
R_{target}
\in
[\min(\mu_i),\max(\mu_i)]
$$

which in this dataset was approximately:

$$
[8.78\%,67.18\%]
$$

Thirty target-return points were evaluated.

The resulting frontier shows the trade-off between expected return and
minimum achievable volatility.

Conceptually:

$$
\boxed{
\text{Efficient Frontier}
=
\text{minimum risk for each feasible target return}
}
$$

The equal-weight portfolio lies away from the minimum-risk region
because equal weighting does not explicitly optimize the covariance
structure.

------------------------------------------------------------------------

# 13. Mean-Variance Optimization

## 13.1 Objective

The classical mean-variance objective used in the notebook was:

$$
\boxed{
\max_w
\left(
w^T\mu
-
\lambda w^T\Sigma w
\right)
}
$$

Because `scipy.optimize.minimize` performs minimization, the equivalent
implemented objective was:

$$
\boxed{
\min_w
\left(
\lambda w^T\Sigma w
-
w^T\mu
\right)
}
$$

where:

-   $w^T\mu$ is expected portfolio return;
-   $w^T\Sigma w$ is portfolio variance;
-   $\lambda$ is the risk-aversion parameter.

The constraints remained:

$$
\sum_iw_i=1
$$

and:

$$
w_i\geq0
$$

## 13.2 Interpretation of $\lambda$

The parameter $\lambda$ controls the relative penalty assigned to risk.

A larger $\lambda$ increases the importance of minimizing variance.

A smaller $\lambda$ puts relatively more emphasis on expected return.

The notebook evaluated:

$$
\lambda\in\{0.1,1,5,10\}
$$

The resulting full-sample portfolios were:

    $\lambda$   Expected Return   Volatility   Sharpe
  ----------- ----------------- ------------ --------
          0.1            67.18%       51.77%    1.298
            1            64.99%       49.52%    1.312
            5            26.84%       22.43%    1.196
           10            18.83%       19.64%    0.959

The plot confirms the expected qualitative effect: increasing risk
aversion moves the solution toward lower-volatility/lower-return
portfolios.

The results also demonstrate that changing the objective parameter can
substantially change portfolio characteristics.

------------------------------------------------------------------------

# 14. Monte Carlo Simulation Fundamentals

The project includes two levels of Monte Carlo analysis.

## 14.1 Basic Monte Carlo convergence experiment

First, a standard normal distribution was simulated:

$$
X\sim N(0,1)
$$

For $N$ independent samples, the sample mean estimates:

$$
E[X]
$$

through:

$$
\boxed{
\hat{E}[X]
=
\frac{1}{N}
\sum_{i=1}^{N}X_i
}
$$

The experiment used:

$$
N\in\{100,1000,10000,100000\}
$$

The results were:

    Simulations   Sample Mean   Sample Std.
  ------------- ------------- -------------
            100       -0.0503        0.7728
          1,000       -0.0186        1.0004
         10,000       -0.0069        1.0090
        100,000       -0.0040        1.0033

The estimates become more stable as the number of simulations increases,
illustrating the basic Monte Carlo convergence principle.

For standard Monte Carlo estimation, sampling error generally decreases
at approximately:

$$
O\left(\frac{1}{\sqrt{N}}\right)
$$

------------------------------------------------------------------------

# 15. Multivariate Monte Carlo Portfolio Simulation

The portfolio-level simulation assumes a multivariate normal model:

$$
\boxed{
R\sim N(\mu,\Sigma)
}
$$

For the final Monte Carlo experiment, the model parameters were
estimated **only from the 2019--2022 training period**.

The annualized parameters were converted to daily quantities:

$$
\mu_{daily}
=
\frac{\mu_{annual}}{252}
$$

and approximately:

$$
\Sigma_{daily}
=
\frac{\Sigma_{annual}}{252}
$$

Then 10,000 multivariate return scenarios were generated:

$$
R_1,R_2,\ldots,R_{10000}
$$

Each simulated observation contained eight correlated asset returns.

The resulting matrix had shape:

$$
10000\times8
$$

------------------------------------------------------------------------

# 16. Simulated Portfolio Returns

For each simulated asset-return vector $R_s$, the portfolio return was
calculated as:

$$
\boxed{
R_{p,s}=w^TR_s
}
$$

This was done for all three portfolio constructions.

Therefore the Monte Carlo experiment produced:

-   10,000 Equal-Weight portfolio outcomes;
-   10,000 Minimum-Variance portfolio outcomes;
-   10,000 MVO portfolio outcomes.

------------------------------------------------------------------------

# 17. Monte Carlo Results

The simulated daily-return distributions were:

  -----------------------------------------------------------------------------
  Portfolio     Mean Daily        Daily          5th         95th   Probability
                    Return   Volatility   Percentile   Percentile       of Loss
  ----------- ------------ ------------ ------------ ------------ -------------
  Equal            0.0831%      1.7678%     -2.8202%      2.9799%        47.98%
  Weight                                                          

  Minimum          0.0365%      1.2153%     -1.9535%      2.0329%        48.69%
  Variance                                                        

  MVO              0.1651%      2.4484%     -3.8474%      4.1844%        47.56%
  -----------------------------------------------------------------------------

The distributions reflect the different portfolio exposures.

The Minimum-Variance portfolio has the narrowest simulated distribution,
consistent with its construction objective.

The MVO portfolio has the widest simulated distribution, reflecting its
greater exposure to the higher-return/high-volatility component of the
estimated return distribution.

The 5th percentile gives a simple downside-tail summary:

$$
Q_{0.05}
=
\text{5th percentile of simulated portfolio returns}
$$

The probability of a negative simulated return was estimated as:

$$
\boxed{
P(R_p<0)
\approx
\frac{
\#\{R_{p,s}<0\}
}{
N
}
}
$$

------------------------------------------------------------------------

# 18. Out-of-Sample Evaluation

A major methodological improvement in the notebook was the final
train/test split.

The data was divided into:

$$
\boxed{
2019\text{--}2022
\rightarrow
\text{parameter estimation and optimization}
}
$$

and:

$$
\boxed{
2023
\rightarrow
\text{out-of-sample evaluation}
}
$$

The training set contained:

-   1,007 daily observations

The test set contained:

-   250 daily observations

The portfolio weights were estimated using only the training data.

The 2023 data was then used only to calculate realized portfolio
performance.

This prevents the final evaluation from directly using the same
observations that generated the portfolio weights.

------------------------------------------------------------------------

# 19. Out-of-Sample Performance

For each portfolio, annualized return was calculated as:

$$
\boxed{
R_{annual}
=
\left(
\prod_{t=1}^{T}(1+R_t)
\right)^{252/T}
-1
}
$$

Annualized volatility was:

$$
\boxed{
\sigma_{annual}
=
\sigma_{daily}\sqrt{252}
}
$$

With a zero risk-free-rate assumption:

$$
\boxed{
Sharpe
=
\frac{R_{annual}}{\sigma_{annual}}
}
$$

Portfolio value was initialized at 100:

$$
V_t
=
100
\prod_{s=1}^{t}(1+R_s)
$$

Maximum drawdown was calculated from the running maximum:

$$
M_t=\max_{s\leq t}V_s
$$

and:

$$
DD_t
=
\frac{V_t}{M_t}-1
$$

Therefore:

$$
\boxed{
MDD=\min_t DD_t
}
$$

The 2023 results were:

  -----------------------------------------------------------------------
  Portfolio    Annual Return         Annual         Sharpe        Maximum
                                 Volatility                      Drawdown
  ----------- -------------- -------------- -------------- --------------
  Equal               78.63%         19.85%          3.960        -10.06%
  Weight                                                   

  Minimum              6.03%         14.23%          0.423        -13.96%
  Variance                                                 

  MVO                118.96%         28.11%          4.231        -14.20%
  -----------------------------------------------------------------------

The cumulative-performance plot shows a substantial divergence between
the portfolios during 2023. The MVO portfolio experienced the largest
growth over the test period, while the Minimum-Variance portfolio
remained much closer to its starting value.

These are **sample-period observations**, not general conclusions about
the methods.

------------------------------------------------------------------------

# 20. Interpreting the 2023 Results

The results illustrate an important portfolio-optimization principle:

> **Risk minimization and return maximization are different
> objectives.**

The Minimum-Variance portfolio had the lowest realized annualized
volatility among the three portfolios:

$$
14.23\%
$$

but also a much lower realized annualized return:

$$
6.03\%
$$

The MVO portfolio had a higher realized annualized return:

$$
118.96\%
$$

but also higher realized volatility:

$$
28.11\%
$$

The Equal-Weight portfolio fell between these two in realized volatility
and return.

The results therefore provide a concrete demonstration of the
risk-return trade-off that motivates mean-variance optimization.

------------------------------------------------------------------------

# 21. Important Research Interpretation

The analysis should **not** be interpreted as:

> "MVO is better than Minimum Variance."

A more precise conclusion is:

> Within this particular eight-asset universe, parameter estimation
> period, long-only constraint set, MVO parameterization, and 2023
> evaluation period, the MVO portfolio produced higher realized return
> and Sharpe ratio than the Equal-Weight and Minimum-Variance
> portfolios, while also experiencing higher realized volatility and a
> slightly larger maximum drawdown.

Similarly:

> The Minimum-Variance portfolio achieved the lowest realized volatility
> in the 2023 test period, but its realized return was substantially
> lower.

This is the correct quantitative-research framing because the experiment
compares objective functions under a fixed experimental setup rather
than establishing a universal winner.

------------------------------------------------------------------------

# 22. Relationship Between Monte Carlo and Out-of-Sample Results

The Monte Carlo simulation and 2023 backtest answer different questions.

### Monte Carlo asks:

> Under the assumed multivariate-normal return model estimated from
> 2019--2022, what does the distribution of one-day portfolio returns
> look like?

### Backtesting asks:

> What actually happened to the portfolio when the fixed weights were
> applied to the unseen 2023 market data?

The two should not be expected to match exactly.

For example, the Monte Carlo model produced an MVO 5th-percentile daily
return of:

$$
-3.85\%
$$

while the actual 2023 path reflects the realized sequence of market
returns rather than simulated draws.

This distinction is fundamental:

$$
\boxed{
\text{Simulation} \neq \text{Historical Backtest}
}
$$

Simulation depends on model assumptions, whereas backtesting uses
realized historical observations.

------------------------------------------------------------------------

# 23. Why the Monte Carlo Model Is Only a Baseline

The simulation assumes:

$$
R\sim N(\mu,\Sigma)
$$

This is useful for learning and as a baseline model, but financial
returns can exhibit features that a simple multivariate normal model
does not capture well, such as:

-   heavy tails;
-   volatility clustering;
-   changing correlations;
-   time-varying expected returns;
-   regime changes.

Therefore the Monte Carlo results should be interpreted as **model-based
scenario analysis**, not as a complete representation of market risk.

A more advanced research project could replace the normal model with
empirical bootstrapping, factor-based simulation, volatility models, or
other distributional assumptions.

------------------------------------------------------------------------

# 24. Key Research Findings

## Finding 1 --- Covariance materially affects portfolio construction

The assets exhibited different correlations, ranging approximately from
0.23 to 0.76.

Therefore portfolio risk cannot be understood by looking at individual
volatility alone.

The covariance matrix is essential:

$$
\sigma_p^2=w^T\Sigma w
$$

------------------------------------------------------------------------

## Finding 2 --- Minimum-variance optimization can create concentrated portfolios

The long-only minimum-variance solution assigned most of its capital to
a small subset of the universe.

This occurs because the optimization objective is solely:

$$
\min_w w^T\Sigma w
$$

with no explicit diversification, maximum-weight, turnover, or
concentration penalty.

This is a practical motivation for adding realistic constraints to
portfolio optimization.

------------------------------------------------------------------------

## Finding 3 --- Risk aversion materially changes MVO behavior

Changing:

$$
\lambda
$$

from 0.1 to 10 changed the estimated portfolio from a
high-return/high-volatility solution toward a
lower-return/lower-volatility solution.

This demonstrates that $\lambda$ is not merely a technical parameter; it
encodes the trade-off between return and risk in the objective.

------------------------------------------------------------------------

## Finding 4 --- Out-of-sample behavior can differ substantially from optimization characteristics

The Minimum-Variance portfolio was explicitly constructed to minimize
estimated variance, and it did produce the lowest realized 2023
volatility among the three portfolios.

However, its realized return was substantially lower.

This demonstrates why portfolio research must evaluate multiple
dimensions rather than relying on a single metric.

------------------------------------------------------------------------

## Finding 5 --- Monte Carlo provides a distribution rather than a single prediction

Instead of producing one expected outcome, Monte Carlo generates a
distribution:

$$
\{R_{p,1},R_{p,2},\ldots,R_{p,N}\}
$$

From this distribution we can estimate quantities such as:

-   mean return;
-   volatility;
-   quantiles;
-   probability of loss.

This makes Monte Carlo useful for studying uncertainty and scenario
behavior.

------------------------------------------------------------------------

# 25. Methodological Limitations

Several limitations should be acknowledged.

### 25.1 Expected returns are estimated using historical averages

The model assumes that the historical sample mean is a useful estimate
of expected return:

$$
\hat{\mu}
=
\text{historical mean return}
$$

This estimate can be noisy, especially because MVO can be highly
sensitive to expected-return inputs.

### 25.2 Sample covariance is used

The covariance matrix is estimated directly from historical returns:

$$
\hat{\Sigma}
=
\text{sample covariance}
$$

No shrinkage or robust covariance estimator is used.

### 25.3 No transaction costs

The backtest assumes frictionless trading.

There is no explicit turnover penalty:

$$
\text{Transaction Cost}=0
$$

### 25.4 No sector or concentration constraints

The optimizer is constrained only by:

$$
\sum_iw_i=1
$$

and:

$$
w_i\geq0
$$

There is no maximum individual weight, sector constraint, or turnover
constraint.

### 25.5 Normal Monte Carlo assumption

The final Monte Carlo experiment assumes:

$$
R\sim N(\mu,\Sigma)
$$

which may not capture all empirical properties of financial returns.

### 25.6 Single out-of-sample year

The final test period is 2023 only.

A stronger research design would use rolling or walk-forward evaluation
across multiple test periods.

------------------------------------------------------------------------

# 26. Important Implementation Caveat: Asset Ordering

There is one implementation detail that should be corrected before
presenting this notebook as a polished final project.

The `tickers` list was defined in this order:

``` text
AAPL, MSFT, GOOGL, AMZN, NVDA, META, JPM, JNJ
```

but the columns returned by the downloaded price dataframe were ordered
as:

``` text
AAPL, AMZN, GOOGL, JNJ, JPM, META, MSFT, NVDA
```

The numerical optimization itself used the dataframe's actual column
order, so the calculations were performed consistently with the
underlying arrays.

However, some displayed weight tables were labeled using the manually
defined `tickers` list rather than the actual dataframe column order.
Therefore the displayed asset labels for optimized weights can be
misleading.

For example, the final training-period MVO weight vector is correctly
interpreted according to the dataframe order:

``` text
AAPL   ≈ 56.54%
AMZN    ≈ 0.00%
GOOGL   ≈ 0.00%
JNJ     ≈ 0.00%
JPM     ≈ 0.00%
META    ≈ 0.00%
MSFT    ≈ 0.00%
NVDA   ≈ 43.46%
```

The correct practice is to derive labels directly from the data:

``` python
asset_names = returns.columns

weights_df = pd.DataFrame({
    "Asset": asset_names,
    "Weight": weights
})
```

This prevents silent mislabeling when numerical arrays are converted
back into human-readable tables.

This is an important reproducibility and research-quality issue.

------------------------------------------------------------------------

# 27. Overall Workflow

The complete project can be summarized as:

$$
\boxed{
\text{Market Prices}
\rightarrow
\text{Daily Returns}
\rightarrow
\text{Return Statistics}
\rightarrow
\text{Correlation/Covariance}
}
$$

followed by:

$$
\boxed{
\text{Equal Weight}
\rightarrow
\text{Minimum Variance}
\rightarrow
\text{Efficient Frontier}
\rightarrow
\text{MVO}
}
$$

and finally:

$$
\boxed{
\text{2019--2022 Estimation}
\rightarrow
\text{Fixed Portfolio Weights}
\rightarrow
\text{2023 Out-of-Sample Test}
}
$$

with a parallel scenario-analysis branch:

$$
\boxed{
(\hat{\mu},\hat{\Sigma})
\rightarrow
\text{Multivariate Monte Carlo}
\rightarrow
\text{Portfolio Return Distributions}
}
$$

------------------------------------------------------------------------

# 28. What This Lab Demonstrates

The completed notebook demonstrates practical understanding of:

-   financial price and return data;
-   return normalization and visualization;
-   return and volatility estimation;
-   correlation and covariance analysis;
-   portfolio return calculation;
-   portfolio variance calculation;
-   diversification;
-   long-only portfolio constraints;
-   constrained numerical optimization;
-   minimum-variance portfolios;
-   efficient frontiers;
-   mean-variance optimization;
-   risk-aversion parameters;
-   Monte Carlo simulation;
-   multivariate return simulation;
-   out-of-sample testing;
-   Sharpe ratio;
-   maximum drawdown;
-   quantitative result interpretation;
-   limitations and model assumptions.

The project therefore serves as a compact introduction to the workflow
used in quantitative portfolio research:

$$
\boxed{
\text{Data}
\rightarrow
\text{Model}
\rightarrow
\text{Optimization}
\rightarrow
\text{Simulation}
\rightarrow
\text{Out-of-Sample Evaluation}
\rightarrow
\text{Research Interpretation}
}
$$

------------------------------------------------------------------------

# 29. Final Conclusion

This lab demonstrates that portfolio construction is fundamentally an
optimization problem built on estimated return and covariance inputs.

The central mathematical object is:

$$
\sigma_p^2=w^T\Sigma w
$$

while mean-variance optimization introduces the return-risk trade-off:

$$
\max_w
\left(
w^T\mu-\lambda w^T\Sigma w
\right)
$$

The empirical results show that changing the optimization objective and
risk preference materially changes portfolio characteristics. The
minimum-variance portfolio produced lower volatility but substantially
lower 2023 realized return, while the MVO portfolio produced higher
realized return and Sharpe ratio alongside higher volatility and
drawdown in this particular test period.

Monte Carlo simulation then provided a complementary perspective by
generating 10,000 hypothetical correlated return scenarios and examining
the resulting distributions.

The most important research lesson is not that one portfolio
construction method is universally superior. Rather, it is that
**portfolio optimization is highly dependent on estimated inputs,
objective functions, constraints, and evaluation methodology**.

That observation provides the natural motivation for more advanced
research into:

-   robust covariance estimation;
-   estimation-error control;
-   alternative objective functions;
-   concentration and turnover constraints;
-   downside-risk objectives;
-   transaction costs;
-   walk-forward validation;
-   and more realistic Monte Carlo models.

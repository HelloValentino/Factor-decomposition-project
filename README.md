# Factor Exposure & Performance Attribution Engine (Global Equity Portfolio)

## Goal (per brief)
Build a simple factor exposure and performance attribution engine for a **global equity buy-and-hold portfolio** using only **public data**:
- Construct a **monthly portfolio return series** from **2016-01 to latest**
- Run a **rolling regression** of portfolio **excess returns** on **Fama-French 5 factors + Momentum**
- Decompose long-run excess return into **factor contributions** and **residual/alpha**
- Produce clear plots + exports

---

## Portfolio assumptions
- **Funded:** 2 Jan 2016  
- **Trading rule:** buy-and-hold (no rebalancing, no flows, ignore costs)
- **Frequency:** daily prices → **monthly** returns for analysis
- **Currency:** USD as implied by Yahoo Finance pricing conventions (see limitations)

---

## Data sources
### Prices
- **Yahoo Finance** via `yfinance` (`auto_adjust=True` to incorporate splits/dividends).

### Factors and risk-free rate
- **Kenneth French Data Library** (monthly, USD):
  - **FF5:** `Mkt-RF, SMB, HML, RMW, CMA`
  - **Momentum:** `MOM`
  - **Risk-free rate:** 1-month T-bill (`RF`)

---

## Methodology

### 1) Monthly returns
- Download daily adjusted prices from Yahoo Finance.
- Resample to **month-end** prices and compute monthly simple returns.

### 2) Missing prices / IPO dates (how gaps are handled)
This implementation uses a conservative, economically interpretable approach:

- **Tickers with no usable Yahoo price history:** excluded from the investable set.
- **Intermittent missing monthly returns (e.g., IPO later / data gaps):**
  - Missing monthly returns are filled with that month’s **risk-free rate (RF)**.
  - Interpretation: if the stock can’t be held/priced in that month, that slice of the portfolio is treated as **cash earning T-bills**.
  - Implication in excess returns: `RF − RF = 0` excess return, which is consistent with “no risk exposure” during missing months.

### 3) Portfolio return series (buy-and-hold)
- Portfolio monthly return is computed as the **weighted sum** of constituent monthly returns.
- Weights are normalized to sum to 1 over available tickers (if some tickers have no data at all).

### 4) Factor model (excess returns)
We estimate, monthly:

\[
(R_{p,t} - R_{f,t}) = \alpha_t + \beta_t^\top F_t + \varepsilon_t
\]

Where \(F_t\) includes: `Mkt-RF, SMB, HML, RMW, CMA, MOM`.

### 5) Rolling regression
- Rolling OLS via `statsmodels.RollingOLS`
- **Window:** 36 months  
  - Rationale: common in applied factor work; balances stability with responsiveness to drift in buy-and-hold portfolios.

### 6) Performance attribution (long-run)
Over the full sample:
- Factor contribution ≈ **average loading × average factor return**
- Residual is captured by the regression **alpha** (and any remaining unexplained component)

---

## Key results (from exported outputs)

### Sample & performance summary
From `summary_statistics.csv`:
- **Period:** 2016-01 to 2025-10 (**118 months**)
- **Positions:** 34
- **Average monthly return:** **1.67%**
- **Annualized return:** **22.05%**
- **Cumulative total return:** **+539%** (growth of $1 ≈ $6.39)
- **Annualized volatility:** **14.78%**
- **Sharpe ratio (annualized):** **1.22**
- **Max drawdown:** **-23.37%**
- **Alpha (annualized, from rolling model average):** **8.80%**

Interpretation (plain English):
- The portfolio delivered **strong absolute and risk-adjusted returns** over the sample.
- Volatility is meaningful (this is not a defensive strategy), but Sharpe > 1 indicates returns have been **good relative to risk**.
- Max drawdown around ~23% indicates the portfolio experiences **material equity-like drawdowns**.

### Return behavior (distribution + time series)
From `return_distribution.png`:
- Monthly return distribution is centered around a positive mean (~**1.6%**).
- There are notable negative tail months (e.g., the large drawdown month around early 2020), consistent with equity crash exposure.
- Returns cluster over time (regimes), which motivates time-varying exposure estimation.

### Average factor exposures (36M rolling)
From `portfolio_analysis_results.csv` (rolling betas averaged over available windows):
- **β(Mkt-RF): ~0.75** → meaningful market exposure, but less than 1
- **β(SMB): ~0.04** → small size tilt is minimal
- **β(HML): ~0.08** → mild value tilt
- **β(RMW): ~-0.08** → mild negative profitability tilt
- **β(CMA): ~0.00** → near-neutral investment tilt
- **β(MOM): ~-0.11** → mild negative momentum tilt

Interpretation:
- The portfolio behaves like a **moderate-beta global equity portfolio** with relatively modest style tilts.
- Exposures vary through time (see plots), so attribution is best interpreted as a **long-run average** plus **time-varying dynamics**.

---

## Plots produced by the notebook
- `factor_loadings_timeseries.png` — rolling 36-month factor loadings over time
- `performance_attribution.png` — bar chart of annualized contribution by factor + alpha
- `cumulative_returns.png` — cumulative growth of $1 vs a market benchmark proxy
- `return_distribution.png` — monthly return histogram + time series

---

## Exported files
- `summary_statistics.csv` — top-line performance metrics and risk stats
- `portfolio_analysis_results.csv` — time series of:
  - portfolio return, excess return, cumulative return, RF
  - rolling betas (`Beta_*`)
  - rolling alpha

---

## How to run (Google Colab)
1. Open the notebook in Colab
2. Run cells top-to-bottom
3. Upload `portfolio.csv`/`.xlsx` when prompted (must include **Ticker** and **Weight**)

Required packages are installed in-notebook (e.g., `yfinance`).

---

## Repo structure (recommended)

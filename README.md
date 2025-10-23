# Quantitative Risk Analysis of LTCM Collapse

A study of why Long-Term Capital Management risk model failed in 1998. The project builds a leveraged convergence portfolio with Bloomberg market data (1995–1998), compares Normal vs. fat-tailed risk models, validates them with backtesting, and reports the breakdown of diversification in crisis.

- Main analysis: [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb)
- Data preparation: [project01_data_analysis.ipynb](project01_data_analysis.ipynb)


figures are generated into the figs/ folder when you run the notebooks.

---

## What this project does

- constructs a market-neutral convergence strategy with high leverage (25x).
- Compares three tail-risk models at 99% confidence:
  - Parametric VaR/CVaR under Normal
  - Monte Carlo VaR/CVaR using fitted Student’s t (fat tails)
  - Historical VaR/CVaR from realized returns
- Validates models via VaR backtesting and Q-Q plots.
- Analyzes crisis dynamics: correlation breakdown, spread divergence, regime shifts.

---

## Data

- Source: Bloomberg Terminal time series, used into dataset/LTCM_database. (data is not published online)
- Period used for replication: 1995-01-02 to 1998-12-31 (1,011 trading days).
- Key series include:
  - Credit spreads (High-Yield OAS), Treasury indices (20+Y), Emerging Markets (JPM EMBI), VIX, 3M Treasury yield
- See [project01_data_analysis.ipynb](project01_data_analysis.ipynb) for Analysis, normalization, and Parquet export.

---

## Portfolio construction

Weights (approximate replication of LTCM’s convergence bet):
- Credit (Bloomberg Aggregate): +50%
- Treasury (20+Y): −30%
- Emerging Markets (EMBI): +30%
- Volatility (VIX): −50%
- Leverage: 25x

Core formulas:
- Unlevered portfolio return: $R_p = \sum_i w_i R_i$
- Levered return with financing: $$R_{p,L} = L \cdot R_p - (L - 1) \cdot R_f$$

Implementation lives in [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb) (Section “Define the Leveraged Portfolio”).

---

## Risk models

1) Parametric (Normal)
- $VaR_\alpha = |\mu + \sigma Z_\alpha|$
- $CVaR_\alpha = \left|\mu + \sigma \frac{\phi(Z_\alpha)}{1 - \alpha}\right|$

2) Monte Carlo (Student’s t)
- Fit $\nu,\ \mu,\ \sigma$ to returns, simulate 100,000 draws
- Compute $VaR_\alpha,\ CVaR_\alpha$ from simulated left tail

3) Historical Simulation
- Percentile and tail mean from realized returns

All three are implemented and compared in [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb) (STEP 2).

---

## Key results (99% confidence)

- Normal VaR: 169.45% | CVaR: 190.80%
- Student’s t VaR: 190.82% | CVaR: 261.27% (ν ≈ 4.38 → very fat tails)
- Historical VaR: 214.04% | CVaR: 284.82%
- Actual maximum single-day loss: 482.84%

Interpretation:
- Normal VaR underestimated t-MC VaR by 12.6% and t-MC CVaR by 36.9%.
- The worst realized loss was ~2.8x the Normal VaR estimate.
- Even the t-distribution improved estimates but still fell short of the realized tail.

Backtesting (expected ≈ 10 violations at 99%):
- Normal VaR: 32 violations (failed)
- t-MC VaR: 18 violations (better)
- Historical VaR: 11 violations (closest)

These outputs are reproduced in [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb).

---

## Crisis dynamics (why diversification failed)

- Spread divergence: Credit and EM spreads widened together in 1998 (Russia default).
- Correlation breakdown: Relationships seen in 1995–1997 did not hold in 1998; correlations moved toward one during stress.
- Regime shift: Rolling volatility, skewness, and kurtosis all worsened into the crisis.
- Short volatility exposure amplified losses.

See “Spread Divergence,” “Rolling Risk Metrics,” and “Correlation Heatmap” sections in [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb).

---

Notes:
- The notebook filters the LTCM window and aligns series by date.
- All figures are saved to figs/ but are not embedded here.

---

## Repository structure

- [project02_quant_analysis.ipynb](project02_quant_analysis.ipynb) — main analysis and visualizations
- [project01_data_analysis.ipynb](project01_data_analysis.ipynb) — data ingest and export helpers
- dataset/ — Bloomberg-derived data workbook (LTCM_database.xlsx)
- figs/ — generated figures (created by the analysis)

---

## What to take away

- Don’t assume normality for leveraged portfolios. Fat tails matter.
- $CVaR$ is a more informative tail metric than $VaR$ for capital planning.
- Correlations drift and often spike toward one under stress.
- Short volatility plus leverage is dangerous in a fat-tailed world.

---

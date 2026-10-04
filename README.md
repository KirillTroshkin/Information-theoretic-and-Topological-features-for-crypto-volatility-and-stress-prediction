# Information-Theoretic and TDA Features for Crypto Volatility and Stress Prediction

**Question:** Is it worth adding cross-asset dependence features to a crypto
volatility model, or is realized volatility (HAR-RV) enough?

**Answer:** For ETH, DOGE, ADA and AVAX, modestly yes (ΔR² of +0.12 to +0.23 at a
2-day feature window). For BTC, SOL and BNB, no: HAR-RV stays better. Averaged over
eight assets there is the gain over a volatility baseline.

## What was tested
- **Features:** rolling mutual information, KL divergence and third/fourth-order
  co-information across assets (information-theoretic, "IT"); persistence-diagram
  summaries of mutual-information complexes (topological, "TDA").
- **Tasks:** forecasting log forward realized volatility (1-14 days) and classifying
  stress events (future volatility above its Q60-Q90 quantile).
- **Configurations:** A = volatility baseline, B = A + IT, C = A + TDA, D = A + IT + TDA.
  Features are added on top of the baseline's residuals; ΔR² and ΔROC-AUC are measured
  against A.

## Setup
- Data: Binance 30-min bars, 8 assets (BTC, ETH, SOL, BNB, XRP, DOGE, ADA, AVAX), 2023-2025
- Split: train until 2025-07-01, purge gap of 96 bars, holdout 2025-07-01 to 2025-09-01
- Models: ridge on baseline residuals (volatility), logistic regression (stress);
  baseline = ridge on the top-3 realized-volatility features
- Benchmarks: HAR-RV and GARCH(1,1)
- Feature selection: permutation importance on the training set

## Results (holdout, mean over assets and horizons)
| Task | Configuration | Result |
|---|---|---|
| Volatility, W=192 | A + TDA (C) | ΔR² +0.054, better in 32/48 tasks |
| Volatility, W=192 | A + IT + TDA (D) | ΔR² +0.040, better in 31/48 tasks |
| Volatility, W=192 | A + IT (B) | ΔR² -0.029, better in 21/48 tasks |
| Volatility, W=96 | A + IT + TDA (D) | ΔR² +0.085, better in 33/48 tasks |
| Stress, W=192 | A + IT + TDA (D) | ΔROC-AUC +0.023; Q80 +0.040, Q90 -0.007 |
| Volatility, W=192, 9 assets incl. TRX | C / D | ΔR² -0.004 / -0.010 |

## Findings
- **By asset.** Gains are concentrated in ETH, DOGE, ADA and AVAX. BTC, SOL and BNB are
  negative or near zero.
- **By window.** IT features help only with short windows (about 2 days); the gain fades
  as the window grows to 8 days. TDA features are stable across windows.
- **Against HAR-RV (W=96).** Near parity on average (R² 0.273 vs 0.276). The augmented
  models beat HAR-RV for five of eight assets (ETH, DOGE, ADA, AVAX with IT+TDA, XRP with
  TDA) and lose for BTC, SOL and BNB.
- **Stress events.** Combined features help most at moderate thresholds (Q60-Q80) and not
  at the rare Q90 threshold in the main specification.

# Stock Prediction Project — Tata Motors (TATAMOTORS.NS)

## Abstract

**We built a pipeline to predict short‑term price direction of Tata Motors
(TATAMOTORS.NS) using historical OHLCV data, technical feature engineering, and
machine learning (XGBoost as the primary model).** _The objective was to explore whether
standard technical indicators produce predictable signals and to visually communicate
findings. In short: the problem is hard, signals are weak, but the experiment is instructive and
produces useful insights for further research._

**Keywords**

- Stock prediction, feature engineering, XGBoost, RSI, MACD, Bollinger Bands, candlestick,
    backtest

## 1. Introduction

Predicting short-term stock movements is famously difficult. This project documents a
practical attempt: collect price & volume data from Yahoo Finance, build technical features,
train models, evaluate and visualize. The goal is not to sell a 'guaranteed signal' but to learn
what works (and what doesn’t) and package results in a clear, reproducible way.

## 2. Data & Preparation

Data span: historical daily OHLCV for TATAMOTORS.NS (approx. 5 years). We used Close
and Volume as primary inputs and ensured columns were normalized (flattened
multi‑index, numeric coercion). Rows with unavailable rolling-window values were
dropped after feature creation.

Target: binary label indicating whether the close price after a short horizon is higher than
today (experiment used 5‑day forward direction). Note: we experimented with both
next‑day and multi‑day horizons; multi‑day tends to be slightly less noisy.

## 3. Feature Engineering

We created a compact but diverse set of features mixing trend, momentum, volatility and
volume signals. Each is computed from raw price/volume:

1. Price-based: returns (pct change), lagged returns (1,2,5 days).
2. Moving averages: MA5, MA20, MA50 and MA ratios / crossovers.
3. Momentum & Oscillators: RSI (14), MACD, MACD signal.
4. Volatility: rolling std of returns (5,10,20 windows).
5. Bollinger Bands: upper/lower + normalized band width.
6. Volume signals: raw Volume change % and (optionally) OBV for future work.


We dropped NaNs produced by rolling windows and aligned features so X and y are clean
for model training.

## 4. Modeling Approach

Baseline models: Naive Bayes and Logistic Regression were tried first as simple baselines.
Primary model: XGBoost (tree-based classifier). Key training choices:

- Time-aware train/test split (no shuffle) to avoid lookahead/data leakage — realistic for
    backtesting.
- Random shuffles used only as diagnostics (to check for learnable static patterns).
- Evaluation metrics: accuracy, confusion matrix, and a simple strategy backtest
    (cumulative returns).

## 5. Results & Visualizations

Below are the main figures generated in the analysis. Each figure includes a concise caption
and a short interpretation — written in a friendly tone grounded in facts.

**Figure 1 — Candlestick chart (last 30 days) with Predicted Signals**

<img width="2100" height="1500" alt="candlestick_predictions_zoomed" src="https://github.com/user-attachments/assets/52077e8c-4e6f-4f54-b033-c1259ad3ef84" />

**Figure commentary:** Green markers = model predicted 'Up' on that day; red X marks a
wrong prediction. This shows where the model signals clustered and where it failed. You


can visually check how many predicted 'Ups' were followed by upward moves. Spoiler:
many predictions are noisy — the model often misreads short-term wiggles.

**Figure 2 — Confusion Matrix (full test set)**

<img width="2100" height="1500" alt="confusion_matrix_full_test" src="https://github.com/user-attachments/assets/9546b458-7a20-420f-8f49-6665eab35499" />


**Figure commentary:** Shows counts of true positives/negatives and false
positives/negatives. The forward time-split accuracy was ≈ 0.492 (close to random). This
emphasizes that while the model picks up patterns when shuffled, realistic forward
performance is weak — a common outcome for daily direction modeling.


**Figure 3 — Cumulative returns: prediction-driven strategy vs buy-and-hold**

<img width="2100" height="1500" alt="cumulative_returns_strategy_vs_bh" src="https://github.com/user-attachments/assets/b543fb2b-5c79-4470-bb3b-07441b304ce9" />


**Figure commentary:** A simple strategy that takes a position according to model prediction
was simulated. The plot shows cumulative performance on the test index. Here buy-and-
hold outperforms the naive prediction-based strategy — which indicates the model's
signals are not reliably capturing directional alpha.


**Figure 4 — Feature correlation matrix (X_test)**

<img width="2100" height="1500" alt="feature_correlation_heatmap" src="https://github.com/user-attachments/assets/1807eb5e-4606-4d9b-9a57-869e9cc58ed7" />


**Figure commentary:** Heatmap reveals high correlations between moving averages (MA5,
MA20, MA50) and some volatility features. High multicollinearity reduces marginal
information from adding more similar features.


**Figure 5 — Feature importances (top features according to XGBoost)**

<img width="2100" height="1500" alt="feature_importances_topk" src="https://github.com/user-attachments/assets/b21165f1-578f-4139-a517-3041d98f28bf" />


**Figure commentary:** MA-related features and Bollinger width appear near the top — the
model relies heavily on moving-average-derived signals. That supports pruning correlated
features or engineering orthogonal signals.


**Figure 6 — Price with moving averages (context window)**

<img width="2100" height="1500" alt="price_moving_averages" src="https://github.com/user-attachments/assets/7fbf33c7-ab2c-46d2-953f-77d4555ae2fa" />


**Figure commentary:** Overlaying MA5/MA20/MA50 on price helps visually inspect
crossovers and trend bias. Useful for qualitative checks and communicating signals to non-
technical stakeholders.


**Figure 7 — Signals timeline and counts (last 30)**

<img width="2100" height="1500" alt="signals_timeline_counts" src="https://github.com/user-attachments/assets/9fcdb0fa-5ebe-4f36-b12f-176e0d1e63b8" />


**Figure commentary:** Grouped bars and counts show the distribution of predicted vs actual
'Up' signals in the zoom window. Handy to spot bias (model predicts more 'Up' than actuals
in the shown window).

## 6. Discussion (what worked, what didn’t)

- Implementation quality: solid. Data handling, feature engineering, no-shuffle time splits —
all correct. The code is reproducible and well-structured for iteration.
- Why performance was weak: short-term stock moves are noisy; many indicators are
transformations of the same price, so they’re correlated. XGBoost can overfit noise if
features are redundant or dataset is small relative to model complexity.
- Positive takeaways: feature importance and correlation plots provide actionable guidance
— prune correlated MAs, add orthogonal features (news, macro, order flow). Use thresholds
(e.g., only predict >0.5% moves) to reduce label noise.

## 7. Limitations & Risks

- Overfitting risk with tree ensembles on small, highly correlated feature sets.


- Backtest assumptions are simplified (no transaction costs, slippage or realistic execution).
Including these often reduces strategy returns.
- Results are specific to the chosen stock and timeframe — out-of-sample behavior may
differ.

## 8. Next steps (practical roadmap)

7. Short term (fast wins): prune correlated features, add lagged labels, try thresholded
    targets, balance classes.
8. Mid term: incorporate sentiment (news), option-implied vol, and use walk-forward
    validation with rolling retraining.
9. Long term: ensemble multiple model types, perform robust hyperparameter search
    (Optuna), and include realistic transaction modeling.

## 9. Appendix — Reproducing the analysis

Install the dependencies:

pip install yfinance xgboost scikit-learn plotly kaleido python-docx tabulate

Run the notebook cells in order. Use the export cell to produce the PNG/HTML assets; this
report was generated by embedding those PNGs.

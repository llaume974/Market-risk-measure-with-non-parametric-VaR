# Market Risk Project — ESILV (2023–2024)

**Author:** Guillaume Attila  
**Course:** Market Risk (ESILV)  
**Period:** 2023–2024

> This README summarizes the coursework brief and provides a clean structure to implement and deliver the project.  
> **Rules (course context):** teams of two from the same TD (one student if odd count); **packages forbidden**; **no AI-generated content**; deliver **PDF report + code** by **Thu 28 Dec**.

---

## Table of Contents

- [1. Objectives](#1-objectives)  
- [2. Data](#2-data)  
- [3. Question A — Historical VaR (Biweight KDE)](#3-question-a--historical-var-biweight-kde)  
- [4. Question B — VaR Backtest (Exceedances)](#4-question-b--var-backtest-exceedances)  
- [5. Question C — EVT (Pickands, GEV/GPD VaR)](#5-question-c--evt-pickands-gevgpd-var)  
- [6. Question D — Almgren–Chriss Estimation & Liquidation](#6-question-d--almgrenchriss-estimation--liquidation)  
- [7. Question E — Multiscale Portfolio Volatility (Wavelets & Hurst)](#7-question-e--multiscale-portfolio-volatility-wavelets--hurst)  
- [8. Deliverables & Grading Hints](#8-deliverables--grading-hints)  
- [9. Reproducibility](#9-reproducibility)  
- [10. References](#10-references)

---

## 1. Objectives

1. **Measure market risk** with non-parametric **VaR** and **ES** on Natixis daily **returns**.  
2. **Validate** VaR via **out-of-sample exceedances**.  
3. Use **Extreme Value Theory** to model tails and compute **EVT-VaR**.  
4. **Estimate** trading-cost parameters of **Almgren–Chriss** and propose an **hourly** liquidation path.  
5. Compute **multiscale** portfolio volatility from **FX** via **wavelets** and **Hurst** exponents; compare with a **direct** method.

---

## 2. Data

- **Natixis daily prices** (TD1):  
  - **In-sample:** Jan-2015 → Dec-2016 (fit)  
  - **Out-of-sample:** Jan-2017 → Dec-2018 (test)
- **FX dataset** (TD5): three FX rates, equal-weight portfolio for multiscale analysis.

**Returns:** use log-returns $r_t = \ln\!\big(P_t/P_{t-1}\big)$. Center if needed.

---

## 3. Question A — Historical VaR (Biweight KDE)

Estimate a **1-day VaR** at level $\alpha$ (parameter) from a **non-parametric** distribution using the **biweight (quartic) kernel**.

### KDE (from scratch)
Given returns $r_{1:n}$, bandwidth $h>0$, kernel $K$,

$$
\hat f_h(x) = \frac{1}{n h}\sum_{i=1}^n K\!\left(\frac{x-r_i}{h}\right), \qquad
\hat F_h(x) = \int_{-\infty}^{x} \hat f_h(u)\,\mathrm{d}u, \qquad
\mathrm{VaR}_\alpha = \inf\{ x:\hat F_h(x)\ge \alpha \}.
$$

**Biweight kernel (support $\lvert u\rvert\le 1$):**

$$
K(u) = \bigl(1-u^2\bigr)^2\,\mathbf{1}_{\{\lvert u\rvert\le 1\}}
\quad\text{(optionally scaled by a constant factor)}.
$$

**Notes**
- Implement quantile search on $\hat F_h$ (monotone).  
- Choose $h$ via simple rules (e.g., Silverman-style) but **implement the rule yourself**.

---

## 4. Question B — VaR Backtest (Exceedances)

On **2017–2018** returns, compute the **exceedance rate**:

$$
\hat p = \frac{1}{N}\sum_{t=1}^N \mathbf{1}\{ r_t < -\mathrm{VaR}_\alpha \}
\quad\text{(loss convention as needed)}.
$$

Compare $\hat p$ to the **target** $\alpha$ (Kupiec-like intuition). Discuss whether the **KDE VaR** is **calibrated** (under/over-conservative).

---

## 5. Question C — EVT (Pickands, GEV/GPD VaR)

1) **Tail index via Pickands (from scratch).** For upper/lower tails, using order statistics $X_{n-k+1:n}$ etc., implement the **Pickands estimator** of the **extreme value index** $\xi$ (choose $k$ rationale, sensitivity study).

2) **EVT-based VaR.**  
   - **Block maxima (GEV)** or **peaks-over-threshold (GPD)** (state choice).  
   - **GPD VaR** at level $\alpha$ with threshold $u$ (exceedance prob. $p_u$) and parameters $\xi,\beta$:

     $$
     \mathrm{VaR}_\alpha^{\mathrm{GPD}}
     = u + \frac{\beta}{\xi}
     \left[\left(\frac{1-p_u}{1-\alpha}\right)^{\xi} - 1\right],
     \quad (\xi\neq 0).
     $$

   - Show **iid** assumption and discuss its plausibility.

3) **Compare** EVT-VaR with KDE-VaR across $\alpha$ (e.g., 95%, 99%).

---

## 6. Question D — Almgren–Chriss Estimation & Liquidation

**Goal:** estimate **temporary/permanent impact** and **risk** parameters, then design an **hourly** liquidation schedule.

- **Model sketch:** execution cost = **permanent impact** $\gamma x$ + **temporary impact** $\eta v$ + noise; risk term via price variance.  
- **Objective:** trade-off **expected cost** vs **risk**:

  $$
  \min_{\{x_t\}}\; \mathbb{E}[\text{cost}(\{x_t\})] \;+\; \lambda\,\mathrm{Var}[\text{cost}(\{x_t\})],
  \quad \text{s.t. } x_0=Q,\; x_T=0,\; \text{one trade/hour}.
  $$

- **Estimation (from price/volume data):** infer $\gamma,\eta,\sigma$ by regressions on signed returns vs signed volumes (no packages; code your own OLS).  
- **Output:** optimal discrete schedule $x_0,\dots,x_T$; show impact of $\lambda$ (risk aversion).

---

## 7. Question E — Multiscale Portfolio Volatility (Wavelets & Hurst)

**Portfolio:** equal-weight of **three FX** series.

### (a) Wavelet correlations + Hurst scaling
1. **Decompose** each series into scales $d=1,\dots,D$ (discrete wavelet transform, coded by hand).  
2. **Scale-wise correlations:** $R_d$ between wavelet coefficients.  
3. **Volatility scaling** using **Hurst** $H$: $\sigma_d = c\,2^{H d}$ (fit $H$ from log-variance vs scale).  
4. **Covariance at scale $d$:** $\Sigma_d = \mathrm{diag}(\sigma_d)\,R_d\,\mathrm{diag}(\sigma_d)$.  
5. **Portfolio vol at scale $d$:** $\sigma_{p,d}^2 = w^\top \Sigma_d w$, with equal weights $w$.

### (b) Direct method (baseline)
- Compute volatility from the **portfolio price series** at different horizons (e.g., 15-min → 1-week), justify **overlapping vs non-overlapping** returns, annualization, and bias/variance trade-off.

**Compare** (a) vs (b): consistency across scales, sensitivity to microstructure noise.

---

## 8. Deliverables & Grading Hints

- **Report (PDF):** concise method explanations + **all code snippets** relevant to each question **embedded** in the report.  
- **Code:** plain source files per question; clearly **from scratch**.  
- **Validation:** show **figures/tables**: KDE fit, exceedance table, Pickands plot, EVT-VaR vs KDE-VaR, AC schedules, multiscale vol curves.  
- **Discussion:** limitations (sample size, iid assumption, bandwidth choice, threshold $u$, microstructure effects).

---

## 9. Reproducibility

- Fix random seeds; document parameters $(\alpha, h, k, u, \lambda, T)$.  
- Provide a small **Makefile** or script that runs **each question** end-to-end and regenerates the results in `report/figures`.

---

## 10. References

- A. C. Davison & R. L. Smith — Models for exceedances over high thresholds (GPD).  
- S. Coles — *An Introduction to Statistical Modeling of Extreme Values*.  
- Silverman — *Density Estimation for Statistics and Data Analysis* (bandwidth rules).  
- Almgren & Chriss (2001) — Optimal execution with impact and risk.

---

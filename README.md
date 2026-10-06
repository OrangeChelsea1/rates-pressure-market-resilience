# POC03-A --- Rates Pressure × Market Resilience

## Does Cross-Asset Stress Predict Future Equity Downside?

**Status: CLOSED --- Primary hypothesis not supported out of sample**

This project tests whether elevated U.S. Treasury-rate pressure,
combined with weakening market resilience, identifies periods of greater
forward equity downside risk.

The practical objective is **risk sizing / avoiding large losses**, not
predicting next-day SPY direction.

## Research question

> Under elevated Treasury-rate pressure, does weakening market
> resilience help distinguish regimes associated with greater forward
> equity downside?

The preregistered directional hypothesis was:

**High Rates Pressure + Weakening Market Resilience → More Negative
20-Day Forward Adverse Excursion (FAE_20D).**

`FAE_20D` is the minimum SPY return observed over the next 20 trading
days. More negative values indicate greater subsequent downside.

## Sample design

-   Complete analysis sample: **5,380 observations**
-   Analysis period: **2005-01-05 to 2026-09-03**
-   DEV: **2005-01-05 to 2022-12-30**
-   CONFIRM: **2023-01-03 to 2025-12-31**
-   LIVE: **2026-01-02 onward**

The 2026 period is treated as a live case, not a fully untouched
out-of-sample period, because contemporary market conditions contributed
to the original research motivation.

Features were constructed on each source's native calendar before
cross-asset alignment. No forward filling was used.

## Frozen feature set

### Rates Pressure

-   `LevelZ252`
-   `Momentum20BP`
-   `ShockZ60`
-   `DistanceHighVolAdj60`

### Equity Price / Concentration

-   `SPYMomentum20`
-   `SPYDistanceHigh60`
-   `EqualWeightRelative20`

### Credit

-   `HYOASLevel`
-   `HYOASChange20BP`

### Volatility

-   `VIXLevel`
-   `VIXChange20`

### FX / Cross-Asset

-   `DXYChange20`
-   `USDJPYChange20`

SPY price itself is used only to construct the forward outcome label.

## Main findings

### 1. Treasury-rate pressure did not provide a stable monotonic downside signal

None of the four preregistered rate-pressure dimensions showed a stable
monotonic relationship with future `FAE_20D` in DEV. `Momentum20BP`
displayed a notable two-tail pattern: both large rate increases and
large rate decreases were associated with worse downside, suggesting
that extreme rate moves may characterize unstable regimes rather than
provide a simple directional signal.

### 2. Several market-response variables looked strong in DEV

Weak SPY momentum, large distance from recent highs, rapid HY OAS
widening, and elevated VIX were associated with worse future downside in
DEV.

These relationships require an important distinction:

**Risk persistence / damage confirmation is not necessarily early
warning.**

A high VIX or already-damaged SPY may identify an active stress state
without providing advance warning.

### 3. Equal-weight weakness failed out of sample

A DEV-defined bottom-20% threshold for `EqualWeightRelative20` was
frozen at approximately **-0.81%**.

In CONFIRM, the historical threshold triggered roughly **49.5%** of
observations, demonstrating substantial distribution shift. More
importantly, the downside relationship reversed.

The candidate signal did not generalize.

### 4. DEV-developed Credit Early Warning rule also failed out of sample

A secondary rule was developed using DEV diagnostics and then frozen:

-   `HYOASChange20BP > +34 bp`
-   `SPYDistanceHigh60 >= -5.64%`
-   `VIXLevel <= 24.52`

Economic interpretation: credit is deteriorating rapidly while headline
equity price and implied volatility have not yet entered severe stress.

**DEV** - Warning observations: 311 - Frequency: 6.97% - Warning Mean
FAE_20D: **-3.26%** - Warning Median FAE_20D: **-2.69%** - Normal Mean
FAE_20D: **-2.58%** - Normal Median FAE_20D: **-1.45%**

**CONFIRM** - Warning observations: 31 - Frequency: 4.16% - Warning Mean
FAE_20D: **-1.28%** - Warning Median FAE_20D: **-0.62%** - Normal Mean
FAE_20D: **-1.97%** - Normal Median FAE_20D: **-1.30%**

The relationship reversed out of sample.

## Robustness checks

### Non-overlapping 20-offset analysis

Because adjacent `FAE_20D` labels share up to 19 future trading days,
the frozen Credit Early Warning rule was evaluated across 20
approximately non-overlapping offsets.

**DEV** - Mean relationship supported in **18/20** offsets - Median
relationship supported in **17/20** offsets - Average mean difference:
**-0.68 percentage points** - Average median difference: **-1.15
percentage points**

**CONFIRM** - Mean relationship supported in only **5/20** offsets -
Median relationship supported in only **5/20** offsets - Average mean
difference: **+0.82 percentage points** - Average median difference:
**+0.25 percentage points**

The DEV relationship was therefore not simply an artifact of overlapping
labels. The principal problem was out-of-sample instability.

### Independent episode analysis

Consecutive warning days were collapsed into single episodes.

**DEV** - 311 warning days → **75 episodes** - Median duration: **2
days** - Mean duration: **4.15 days** - Mean FAE_20D from first warning:
**-2.25%** - Median: **-1.58%** - Worst: **-15.61%**

**CONFIRM** - 31 warning days → **8 episodes** - Median duration: **2
days** - Mean duration: **3.88 days** - Mean FAE_20D from first warning:
**-0.69%** - Median: **-0.44%** - Worst: **-5.49%**

The DEV effect persisted after collapsing repeated daily warnings, but
again did not generalize to CONFIRM. The small number of CONFIRM
episodes limits precision.

## Final conclusion

### Primary hypothesis: NOT SUPPORTED

The evidence does not support the claim that elevated Treasury-rate
pressure combined with weakening market resilience reliably predicts
greater 20-day SPY downside.

Several economically intuitive relationships were strong in DEV, but
candidate early-warning signals did not survive the untouched 2023--2025
confirmation period.

The study therefore does **not** support mechanically reducing equity
exposure based on: - elevated Treasury yields alone, - extreme
equal-weight underperformance alone, - or the tested fixed Credit Early
Warning rule.

The central research lesson is **non-stationarity / regime dependence**:

> A signal can be economically intuitive and historically strong, yet
> fail when the market regime changes.

A natural next research question is:

> Under what market regimes do rates, credit, volatility, and
> market-internal signals contain useful information about future equity
> downside?

That question is deliberately left for a separate study rather than used
to rescue POC03-A.

## Research integrity

This repository preserves negative results.

After the confirmation failure, the study did **not**: - change the
20-day label horizon, - re-optimize the frozen thresholds on CONFIRM, -
re-quantile the confirmation sample to manufacture a comparable trigger
rate, - remove inconvenient years, - or add a more complex model to
rescue the original hypothesis.

The failed out-of-sample test is the result.

## Repository structure

``` text
POC03_Rates_Pressure_Market_Resilience/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── POC03A_Rates_Pressure_Market_Resilience.ipynb
├── docs/
│   └── FINAL_RESEARCH_RECORD_CN.md
└── data/
    └── README.md
```

## Data sources

See `data/README.md` for data provenance, including the HY OAS
historical-series limitation and archived FRED snapshot used in the
research.

## Disclaimer

This project is for research and educational purposes only. It is not
investment advice.

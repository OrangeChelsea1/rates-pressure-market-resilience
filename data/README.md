# Data Notes and Provenance

This repository does not redistribute third-party market datasets.

The notebook retrieves or reconstructs the research inputs from their
original/accessible sources.

## Main series

-   U.S. 10-Year Treasury Constant Maturity Rate: FRED `DGS10`
-   High Yield Option-Adjusted Spread: FRED `BAMLH0A0HYM2`
-   SPY / RSP / VIX / DXY / USDJPY: Yahoo Finance / `yfinance` symbols
    used in the notebook

## HY OAS historical-data limitation

During the research, the currently accessible FRED `BAMLH0A0HYM2` series
exposed only a shorter recent history because of a licensing/access
change.

To preserve the longer historical research sample, an archived official
FRED CSV snapshot was used for the older period and then merged with the
currently accessible FRED series.

The archived and current series were validated on their overlapping
period: - 546 overlapping observations - overlap period: 2023-10-03 to
2025-11-03 - maximum absolute difference: 0.0 - mean absolute
difference: 0.0

The notebook is the source of truth for the exact retrieval and merge
logic.

## Calendar treatment

Rolling features are calculated on each source's native observation
calendar before cross-asset alignment.

No forward filling is used to manufacture observations on dates where
the underlying series is unavailable.

## Point-in-time caveat

An observation date is not necessarily identical to the exact real-time
availability timestamp. This project is a research prototype and does
not claim full point-in-time/vintage reconstruction for every source.

## Reproducibility caveat

Third-party providers can change historical access, licensing, ticker
behavior, or data revisions over time. A future rerun may therefore
require source updates even when the research logic remains unchanged.

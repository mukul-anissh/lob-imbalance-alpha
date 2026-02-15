# lob-imbalance-alpha
Event driven study on whether top of book volume imbalance predicts short horizon mid price movement and its naive econoomic value.

## Motivation
Modern electronic markets continuously update the limit order book as liquidity is added, removed, or executed.
An imbalance between bid and ask depth may indicate short term pressure on price.

This project performs a first pass empirical investigation using only L1 data.

## Research Question
Does top of book volume imbalance at time t influence the probability and direction of mid price movement over the next k order book updates?
If yes, can this information align with positive mark to mid returns under simplified assumptions?

## Data
Instrument: BTCUSDT
Venue: Binance
Date: 14-02-2024
Data type: Level-1 best bid/ask updates
Time index: event time (row order)
Dataset size: 45.27 million updates

## Preprocessing steps
- Sorted chronologically by event time.
-  Checked for invalid or zero quantities.
-  Computed spread and filtered extreme dislocations (above 99.9%ile) to focus on normal trading conditions.
-  Constructed mid price.
A small sample is provided in sample_data/ to illustrate schema.

## Feature Engineering
#### Mid Price
$$
\text{Mid Price} = \frac{\text{Best Ask Price} + \text{Best Bid Price}}{2}
$$

#### Imbalance
$$
I_t = \frac{Q^{bid}_t - Q^{ask}_t}{Q^{bid}_t + Q^{ask}_t}
$$

where:
- $Q^{bid}_t$ = best_bid_qty at time $t$
- $Q^{ask}_t$ = best_ask_qty at time $t$

#### Horizon
Fixed number of future updates. 

$$
\text{Future Mid}_t = \text{Mid}_{t+k}
$$

## Labels
#### Directional

$$
\text{up} = \text{Future Mid} > \text{Mid}
$$

#### Adverse Selection Proxy

$$
\text{not down} = \text{Future Mid} >= \text{Mid}
$$

## Methodology
The study proceeds in three layers.
### 1) Conditional probability of upward move
Imbalance values are bucketed by quantiles to ensure stable sample sizes.
For each bucket, we estimate:

$$
P(up|imbalance)
$$

### 2) Probability of avoiding adverse movement
We similarly compute:

$$
P(not down∣imbalance)
$$

This is closer to passive trading intuition, though it ignores fill mechanics.

### 3) Naive economic alignment (toy backtest)
We simulate a simple rule:
- long if imbalance in top decile
- short if in bottom decile
- hold for k updates
- evaluate mid to mid

Return:

$$
pos \times (\text{future mid} −\text{mid})
$$

This measures whether price drifts in the signal direction, not real tradable PnL.

## Key Findings

<img width="497" height="463" alt="image" src="https://github.com/user-attachments/assets/a2ac5aca-7931-4337-be0b-b5e5cc993206" />
<img width="497" height="453" alt="image" src="https://github.com/user-attachments/assets/048b6e33-9108-4622-b79d-e0695cadafa5" />

(Ignores spread, fees, and fill mechanics)

- Conditional probabilities exhibit a monotonic relationship with imbalance.
- Extreme buy pressure corresponds to materially higher chance of upward movement relative to unconditional baseline.
- Similar structure appears in adverse-selection probabilities.
- In the toy simulation, directional alignment produces positive cumulative drift across millions of trades.

Given the very large sample size, these effects are statistically stable.

## Interpretation
Imbalance does not strongly predict that price will move.
It instead tilts odds of the direction conditional on movement.
This is typical for microstructure signals.

## Limitations
This project intentionally ignores many realities of execution:
- assumes fills at mid
- no spread crossing or fees
- no queue position
- no latency
- no market impact
- single instrument / limited regime

Therefore results represent informational content, not deployable profitability.

## Future Work
Natural extensions include:
- conditioning on price change events
- horizon or decay analysis
- incorporating spread costs
- fill probability modeling
- regime stability across days
- dynamic threshold selection
- signal combination

## Reproducability
Run notebooks in order:
1. data_cleaning.ipynb
2. analysis.ipynb
3. trading_simulation.ipynb

## Takeaway
This project demonstrates an end to end research pipeline:

$$
\text{data validation} → \text{feature construction} → \text{forward labeling} → \text{conditional statistics} → \text{economic translation}​
$$


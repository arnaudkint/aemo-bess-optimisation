# AEMO BESS Revenue Optimisation Model

A degradation-aware battery energy storage system (BESS) revenue optimisation model for the
South Australian region of Australia's National Electricity Market (NEM).

## What this is

A mixed-integer linear program (MILP) that optimises battery dispatch for energy arbitrage,
maximising revenue net of a degradation cost rather than gross revenue alone. The headline result
compares a degradation-naive dispatch strategy (maximise short-term revenue, ignore cycling cost)
against a degradation-aware one (price in the cost of cycling), across a range of degradation cost
assumptions.

Built on public AEMO five-minute dispatch price data, modelled on a 150MW / 600MWh (4-hour
duration) asset in South Australia, for the 2024 calendar year.

## Why degradation-aware

Most public BESS arbitrage models optimise gross revenue only, which implicitly assumes cycling
the battery has no cost. In reality, every cycle consumes a finite asset life, and a naive
dispatch strategy will over-cycle relative to what maximises lifetime value. This model treats the
degradation cost as a priced term in the objective function, not just a constraint, so the dispatch
decision itself trades off short-term revenue against long-term asset value, the way a real
commercial operator or asset owner actually has to think about it.

## Important limitation: this is a perfect-foresight ceiling, not an achievable forecast

The model assumes full intraday knowledge of each day's five-minute prices at the point of
optimisation. Real dispatch decisions are made ahead of actual prices being known. This is a
deliberate and named simplification, not an oversight, results should be read as an upper bound on
achievable revenue, benchmarked against real-world South Australian BESS performance
(Hornsdale Power Reserve) as a sanity check. A planned extension adds a price-forecasting layer to
quantify the realistic gap between this ceiling and achievable revenue under forecast uncertainty.

## Scope

**In scope (this version):** energy arbitrage net of a linear, throughput-based degradation cost,
South Australia, 2024, five-minute resolution.

**Explicitly out of scope (named future extensions):** FCAS co-optimisation, machine-learning
price forecasting and realistic-bidding analysis, multi-year cannibalisation analysis.

See `AEMO_BESS_Model_SPEC.md` for the full specification, including battery parameters, solver
choice, and the reasoning behind each modelling decision.

## Repository structure

```
data_ingestion/   AEMO data download and cleaning
model/            Pyomo MILP formulation and daily rolling solve logic
scenarios/        degradation-cost sensitivity runs, naive-vs-aware comparison
results/          plotting, revenue decomposition, benchmarking outputs
```

## Stack

Python, Pyomo, HiGHS solver. No commercial solver dependency.

## Disclaimer

Battery parameters used are illustrative, standard grid-scale lithium-ion assumptions chosen to
match a real asset's power and energy rating for narrative purposes. They are not the actual
specification or performance data of any specific commercial asset.

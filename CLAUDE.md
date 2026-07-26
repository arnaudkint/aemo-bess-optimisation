# Project: AEMO BESS Revenue Optimisation Model

Read `AEMO_BESS_Model_SPEC.md` in the repo root first — it is the full one-page specification
and is authoritative for scope, parameters, and modelling decisions. This file is a short pointer
and working-agreement layer on top of it.

## What this project is
A degradation-aware BESS energy arbitrage optimiser for the South Australian NEM region, built as
a public GitHub artifact. It pairs MILP/optimisation capability with BESS technical-performance
depth (RTE loss decomposition work done separately at Luminus). The public-facing narrative is
technical-commercial hybrid, not pure trading, not pure engineering.

## Current phase
Version 1 build: degradation-aware perfect-foresight arbitrage optimiser only. Do not implement
FCAS, ML forecasting, or multi-year analysis unless explicitly asked — these are named, deferred
extensions, not part of the current scope, and pulling them in early is scope creep against a
deliberate decision.

## Non-negotiable modelling decisions (see spec for full detail and rationale)
- Region: South Australia. Year: 2024 (full calendar year). Resolution: 5-minute dispatch prices.
- Objective: maximise arbitrage revenue minus a linear, throughput-based degradation cost.
- Degradation cost is a sensitivity (low/central/high), never a single hardcoded value.
- Daily rolling optimisation with full intraday (288 interval) perfect foresight, SoC carried
  forward across day boundaries — not a full-year single solve.
- Battery parameters (150MW/600MWh, 90% RTE, 10–90% SoC band) are explicitly illustrative,
  literature-sourced assumptions, not Navagne's real confidential specification. Never imply
  otherwise in code comments, README, or output labels.
- Negative prices must be natively supported in the formulation, not treated as an edge case.
- Stack: Pyomo + HiGHS solver. No commercial solver dependency.
- Any interval flagged with a market intervention indicator is excluded from headline results and
  noted separately.

## Data sourcing tasks to do at the start of the build (not yet resolved, do this first)
1. Pull specific low/central/high degradation cost figures (CSIRO GenCost, BloombergNEF, or
   similar public grid-scale lithium-ion cost benchmark).
2. Pull Hornsdale Power Reserve published revenue-per-MW / cycles-per-year figures as the
   benchmark comparison target.
3. Confirm access route for 5-minute AEMO dispatch price data for SA, 2024 (MMSDM Historical Data
   SQLLoader directly, or via NEMOSIS / nem-data Python packages).

## Repo structure (target)
```
/data_ingestion/     - AEMO data download + cleaning (intervention-flag exclusion here)
/model/              - Pyomo formulation, daily rolling solve logic
/scenarios/          - degradation-cost sensitivity runs, naive-vs-aware comparison
/results/            - plotting, revenue decomposition, benchmarking outputs
AEMO_BESS_Model_SPEC.md
README.md            - states scope, assumptions, and the perfect-foresight caveat up front
CLAUDE.md
```

## Working style
Arnaud is an experienced MILP practitioner (KU Leuven thesis on Belgium hydrogen infrastructure
optimisation) but new to Pyomo specifically and to the NEM market structure. Explain Pyomo-specific
syntax choices briefly where non-obvious. Do not over-explain general MILP/optimisation concepts he
already knows. Flag clearly if a request would pull in V2-scope work (FCAS, ML, multi-year) so it
can be deferred deliberately rather than absorbed silently.

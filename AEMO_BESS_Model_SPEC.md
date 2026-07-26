# AEMO BESS Revenue Optimisation Model — Specification

## Purpose
Public GitHub artifact demonstrating a technical-commercial hybrid capability: a degradation-aware
battery energy storage system (BESS) revenue optimisation model for the Australian National
Electricity Market (NEM). Built to translate existing MILP optimisation experience (Belgium
renewable hydrogen infrastructure thesis) into the NEM domain, and to pair commercial revenue
optimisation with technical BESS performance modelling (RTE loss decomposition work at Luminus).

This is Version 1. Version 2 (deferred, not in scope here) adds a gradient-boosting price
forecaster to compare perfect-foresight revenue against realistic, forecast-constrained dispatch.
FCAS co-optimisation and multi-year cannibalisation analysis are also named future extensions,
not part of this build.

## Why this build, and what was rejected

Several alternative builds were considered and rejected before settling on this one.

A pure energy arbitrage optimiser (no degradation) was rejected as a "commodity artifact" — it is
the first thing any candidate with a solver builds, and it says nothing about the intersection of
technical BESS performance knowledge and commercial optimisation that is this candidate's actual
differentiator.

Energy + FCAS co-optimisation was rejected for V1 (though retained as a named V2 extension). FCAS
is more NEM-specific and impressive to a generalist audience, but it is a harder MILP and the data
is messier, and — more importantly — it is not where this candidate's unique edge lies. With a
limited effort budget, the degradation-aware core (unique) is built first; FCAS (broadens NEM
credibility generally) comes second if there's runway.

A pure foresight-vs-realistic-bidding ML study was rejected as the core build (though the ML
forecasting layer is retained as a named V2 extension) because it would pull the whole project
toward forecasting/ML skills the candidate does not yet have, at the expense of the degradation
depth that is the actual differentiator, and risks finishing nothing if attempted first.

A cannibalisation study (revenue/MW compression as storage penetration rises) was rejected as a
standalone core artifact — rigorous treatment requires counterfactual market analysis or
econometrics, which is a research problem, not a clean optimisation deliverable, and uses neither
the MILP strength nor the BESS technical depth that are this candidate's actual assets. It is
retained as a cheap by-product of running the same model across multiple years later, not as the
spine of the build.

The chosen build — a degradation-aware SA arbitrage optimiser — is the only option that sits
directly at the intersection of MILP optimisation capability and BESS technical performance
knowledge (RTE loss decomposition work), which is the exact technical-commercial hybrid
positioning being targeted for the Australia move.

## Region
South Australia (SA), chosen over Queensland. SA has the highest renewable penetration and price
volatility in the NEM, driven by renewable variability rather than QLD's more thermal-outage-driven
volatility, which makes it a more interesting arbitrage signal. SA also has the deepest grid-scale
BESS presence (including Hornsdale Power Reserve, used as the benchmark target below), so results
are benchmarkable against real assets doing the same thing. It is also the region most relevant to
target employers' commercial teams (Fluence, AMPYR, Powerline, Eku), all of whom think about SA
BESS economics specifically.

## Year and resolution
Full calendar year 2024. Chosen over 2022 (contaminated by the June 2022 market suspension,
an active subject of academic correction in NEM literature) and over 2025 (recency without a
clear finalisation/cherry-picking advantage). Five-minute dispatch price data throughout, sourced
from AEMO's MMSDM Historical Data SQLLoader or a wrapper package (NEMOSIS or nem-data). Thirty-
minute data is simply an aggregation of five-minute dispatch prices post-October 2021 and is not
used as a staged fallback; there is no meaningful access barrier to five-minute data.

## Revenue scope
Energy arbitrage only, net of a degradation cost. FCAS explicitly out of scope for V1 (deferred to
a later version) — arbitrage-only would be a commodity artifact; FCAS is a beginner NEM add but not
where this candidate's differentiation lies. Degradation-awareness is the differentiator and is
in scope from V1.

## Objective function
Maximise: total arbitrage revenue (sum of dispatch price × net discharge over each 5-min interval)
minus degradation cost (linear, throughput-based: cost per MWh cycled).

Subject to: state of charge dynamics, state of charge bounds, charge/discharge power limits,
round-trip efficiency.

Degradation enters as a priced term in the objective (not merely a constraint), because the
headline result is the comparison between degradation-naive and degradation-aware dispatch.

Degradation cost is NOT a single point value — run as a sensitivity across low/central/high
estimates drawn from a public benchmark (e.g. CSIRO GenCost, BloombergNEF grid-scale lithium-ion
augmentation/replacement cost). Specific figures to be sourced at the start of the build session
(data lookup, not a modelling decision).

## Battery parameters (illustrative, not Navagne's actual confidential specification)
- Power / energy: 150 MW / 600 MWh (4-hour duration, matches Navagne rating for narrative
  continuity only)
- Round-trip efficiency: 90%
- State of charge operating band: 10–90% of nominal capacity
- Charge/discharge power: symmetric, full 150 MW rating
State explicitly in the README that these are illustrative, literature-standard grid-scale
lithium-ion assumptions, not Navagne's real parameters.

## Time granularity / computation
Daily rolling optimisation, full intraday (5-minute, 288 intervals/day) perfect foresight within
each day, state of charge carried forward sequentially across day boundaries. Chosen over a
full-year single solve: computationally tractable, and more defensible as a "perfect-foresight
ceiling" — next-day price certainty is arguably approximable by a good forecaster (sets up V2
cleanly); year-long certainty is not and would overstate the ceiling.

## Negative prices
SA sees negative pricing regularly. The objective/constraint formulation must natively support
negative regional reference prices from the first version of the code — charging during negative
price periods is expected, economically meaningful behaviour, not an edge case.

## Data quality handling
Any 5-minute interval flagged with a market intervention indicator is excluded from the headline
result and reported separately as a data quality note.

## Solver / stack
Pyomo (over PuLP) — more expressive, standard in energy-systems modelling, and the right
investment given planned extensions (FCAS co-optimisation adds constraint complexity that would
otherwise force a migration). Solver: HiGHS — open source, fast enough for this problem size, no
licensing dependency in a public repo.

## Benchmark target
Hornsdale Power Reserve — best-known SA BESS with publicly reported commercial performance.
Actual published revenue-per-MW / cycling figures to be sourced at the start of the build session.

## Four required outputs
1. Optimal dispatch and state-of-charge profile over representative days
2. Revenue decomposition: gross arbitrage revenue, degradation cost, net revenue
3. Headline comparison: degradation-naive vs degradation-aware dispatch (gross revenue vs cycles/
   lifetime value tradeoff), across the low/central/high degradation cost sensitivity
4. Benchmarking summary: revenue per MW, cycles per year — explicitly framed as a perfect-
   foresight upper bound, sanity-checked against Hornsdale Power Reserve published figures

## Explicitly out of scope for V1 (named future extensions)
- FCAS co-optimisation
- ML/gradient-boosting price forecasting and realistic-bidding gap analysis
- Multi-year cannibalisation analysis (arbitrage revenue/MW decline as spreads compress)

## Honest limitations (state these, don't hide them)

This model produces a perfect-foresight revenue ceiling, not an achievable revenue forecast. Real
dispatch requires bidding ahead of actual prices; this model assumes full intraday knowledge of the
day's five-minute prices at the point of optimisation. This is a known and named limitation, not an
oversight — it is exactly what Version 2's forecasting layer is designed to address, by comparing
this ceiling against a realistic, forecast-constrained dispatch strategy. Reporting the V1 result
explicitly as a ceiling, and naming the realistic-bidding gap as a defined next step, turns this
limitation into evidence of sophistication rather than a hole an interviewer can exploit.

The degradation cost is a modelling simplification (linear, throughput-based) chosen deliberately
over a physics-grade electrochemical or depth-of-discharge-dependent model, which would sink the
build's timeline for limited narrative benefit. This is the standard marginal-cycling-cost approach
used in real commercial BESS bidding and is defensible from the literature; a non-linear,
depth-dependent cost via piecewise linearisation is a clean V2 refinement, not a V1 requirement.

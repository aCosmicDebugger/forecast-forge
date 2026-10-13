# Decision log

Recorded: 2026-10-10.

## ADR-001 — Regional weather forecasting

**Status:** Accepted.

forecast-forge becomes the temporal forecasting extension of cosmic-downscaler for Northeast Brazil. This supersedes the earlier generic time-series laboratory scope. Temporal backtesting remains a first-class subsystem.

## ADR-002 — Common regional grid and five initial fields

**Status:** Accepted.

Use the companion project's versioned nominal 8 km reference grid over longitude −49° to −33° and latitude −19° to +1°. Start with t2m, d2m, u10, v10 and msl. Derive wind speed from components.

The broad box includes ocean and buffers. Administrative reporting uses a separate mask. More variables remain an extension requiring validated data and semantics.

## ADR-003 — Six-hour cadence and 72-hour horizon

**Status:** Provisional MVP default.

Produce twelve leads from +6 to +72 hours. Proposed input history is five frames from −24 hours to the origin.

This bounds initial compute and evaluation complexity. Hourly output is deferred; interpolation would not establish hourly forecast skill. Revisit cadence and history after auditing continuous source availability.

## ADR-004 — Separate generated targets from independent references

**Status:** Accepted.

Historical downscaled GFS fields are candidate training inputs and synthetic targets. Evaluate agreement with them explicitly as a synthetic-field task.

Assess forecasting skill separately against held-out BRAMS and compatible observations when available. BRAMS represents numerical-model output. Neither BRAMS nor synthetic downscaled fields should be labeled observational ground truth.

## ADR-005 — Share an export contract, not internal repository code

**Status:** Accepted.

Connect repositories through exported datasets, schema/grid versions and checkpoint provenance. Avoid imports from another checkout or an implicit runtime dependency on its directory structure.

## ADR-006 — Chronological splitting applies to the entire pipeline

**Status:** Accepted.

The downscaler, forecaster and fitted transforms must exclude held-out target information. Split by time before training either model; keep patches and windows together; purge targets that cross partition boundaries.

An already trained downscaler is usable for this evaluation only if its training provenance demonstrates compatibility with the split.

## ADR-007 — GFS-first causal training and production

**Status:** Accepted; supersedes the ERA5-first operational-source direction.

GFS is the primary historical and operational source. ERA5 remains an optional retrospective experiment. Freeze the selection of available analyses or short-lead states, retaining initialization, source lead, valid time and publication time. Audit archival coverage and source upgrades.

The first Forge model ingests only past/current downscaled regional states. It learns temporal evolution up to 72 hours. Native and downscaled future GFS trajectories are mandatory comparisons, not implicit future inputs. A future-conditioned variant is a separate experiment.

Construct future synthetic targets using the frozen historical state policy, rather than silently treating the origin cycle's long-lead forecasts as observed future states. Independently verify against held-out references.

## ADR-008 — Baselines precede deep learning

**Status:** Accepted.

Implement persistence and daily-cycle persistence before a compact learned model. Select the learned architecture only after data and memory profiling. Diffusion is outside this forecasting MVP.

## ADR-009 — Evaluate every field and lead

**Status:** Accepted.

Report MAE, RMSE and bias by channel and lead, wind-vector/speed errors, geographic diagnostics and paired baseline skill. Do not obscure a deteriorating variable with one aggregate score.

Specify spatial weighting and use temporal blocks for uncertainty estimates. No numeric improvement threshold is fixed before baseline results.

## Open implementation decisions

- Historical period, paired data coverage and final split dates.
- Export/storage format and geographic reporting mask.
- Compact model architecture, training patches and boundary treatment.
- Operational provider, availability policy and source adaptation.
- Availability of observational validation and its representativeness.
- How future stochastic downscaler outputs preserve temporal coherence.

## ADR-010 — GOES and energy remain extensions

**Status:** Accepted.

Maintain the nominal 8 km atmospheric MVP. GOES may complement future models with cloud/convection observations; no all-variable 0.5 km target is adopted. Energy generation, severe-weather guidance and outage risk form a separate future application layer with additional targets, data and validation.

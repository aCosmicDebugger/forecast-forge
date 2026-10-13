# Architecture

## Responsibilities

| Component | Responsibility |
| --- | --- |
| cosmic-downscaler | Spatial refinement, historical export, checkpoint and grid provenance |
| forecast-forge | Temporal prediction, forecast-origin construction, backtesting and evaluation |
| Source adapters | Convert each source into the common contract without hiding availability or forecast lead |

The initial implementation remains a Python package, forecast_forge, with production logic under src/. Notebooks support exploration and visualization.

## Proposed package organization

| Module | Responsibility |
| --- | --- |
| data/ | Source adapters, contract validation, manifests and dataset access |
| features/ | Temporal windows, calendar features and training-only transformations |
| models/ | Baselines and a common learned-model interface |
| backtesting/ | Chronological splits, rolling origins and availability checks |
| evaluation/ | Metrics, baseline comparisons and geographic aggregation |
| visualization/ | Forecast maps, lead-time curves and error diagnostics |

Keep configuration under configs/, experimental notebooks under notebooks/, and metrics/figures under reports/. Large data and checkpoints remain outside Git.

## Versioned data contract

Proposed logical representation: values indexed by sample valid time, channel, latitude and longitude. A forecast dataset additionally records forecast origin and lead. Storage format is selected after profiling; do not freeze a dependency before implementation.

| Field | Requirement |
| --- | --- |
| schema_version | Contract version |
| grid_id | Immutable grid identifier including coordinates and masks |
| channels | Explicit ordered names and canonical units |
| valid_time | UTC time represented by each field |
| initialization_time | Source model initialization, where applicable |
| source_lead_hours | Original numerical-forecast lead, where applicable |
| available_at | Actual publication/ingestion timestamp, or a documented estimate and its uncertainty |
| source | Dataset/provider/model and original file identifiers |
| processing | Downscaler checkpoint, configuration, code version and deterministic seed where applicable |
| quality_mask | Validity/missingness per field and grid cell |
| generation_kind | Reference model, observation or downscaled synthetic field |

Canonical channels:

| Channel | Unit | Meaning |
| --- | --- | --- |
| t2m | K | Temperature at 2 m |
| d2m | K | Dewpoint at 2 m |
| u10 | m/s | Eastward wind component at 10 m |
| v10 | m/s | Northward wind component at 10 m |
| msl | Pa | Mean sea-level pressure |

Map source identifiers explicitly. Validate that pressure is mean sea-level pressure rather than surface pressure. Convert longitudes to a consistent −180° to +180° convention and enforce one latitude orientation and channel order. Do not replace missing values with physical zeros.

Speed is sqrt(u10² + v10²). Predicting components preserves directional information. Report physical-consistency violations, including dewpoint above temperature, rather than silently clipping raw outputs.

## Training path

1. Align GFS and reference fields by valid time and common spatial domain.
2. Split the timeline before fitting the downscaler and forecaster.
3. Train/freeze a downscaler using training data only; export regional historical fields with provenance. Prefer time-blocked out-of-fold exports for Forge training to reduce mismatch between in-sample and held-out downscaler errors. Never use held-out target information to fit either model.
4. Form contiguous history/target windows at six-hour cadence.
5. Fit forecaster transformations and model only on the training partition.
6. Select settings on validation data; evaluate once on held-out test periods.

The proposed input contains five frames over the preceding 24 hours and the origin itself. Targets contain twelve future frames. History length is adjustable after profiling. Missing timestamps invalidate a window unless an explicit, causal imputation policy is evaluated.

Training on generated fields can transfer spatial biases and smooth temporal extremes. Preserve a reference-data comparison and examine time continuity, especially if a stochastic downscaler is introduced.

## Backtesting and leakage controls

Group every grid cell, patch and overlapping window from the same temporal sample into the same split. A sample's complete target interval must stay inside its partition; purge origins whose targets cross a boundary. Context from an earlier partition is allowed only if it would have been available at forecast time.

Historical GFS experiments must replay actual operational availability. ERA5 is an optional retrospective experiment and cannot substitute for timely GFS inputs without separately evaluated adaptation.

For operational backtests, every input must satisfy available_at ≤ forecast issuance. Never substitute a later model cycle or revised analysis. Retain source lead so states from different forecast leads are not silently treated as identical initial-condition products.

Retraining schedules, preprocessing versions, source policies and checkpoint selection must be recorded per backtest run.

## Model progression and resource budget

First implement persistence: repeat the field at the origin at every lead. Daily-cycle persistence repeats the most recent observed 24-hour pattern across the horizon using only pre-origin frames.

Then benchmark a compact learned model with explicit spatial context and temporal history. Decide between a convolutional temporal model and another small architecture from measured memory, runtime and validation results.

Profile batch size, history, channels and spatial extent before training. Use patches with halos or reduced-domain experiments if needed, validating edge effects and spatial stitching. A 16 GB GPU is a design constraint, not a guarantee of full-domain feasibility.

## Operational path

Primary chain: GFS recent states → cosmic-downscaler regional fields → forecast-forge future regional fields. The Forge predicts temporal evolution; it does not merely return the future trajectory already produced by GFS.

Freeze the rule for selecting analyses or short-lead forecasts available at each origin, recording source cycle, lead and publication time for every history frame. Use the same rule for historical window construction and production. A forecast-derived state is not an observation.

The first Forge model receives no future-valid GFS fields. Native GFS and downscaled future GFS trajectories provide mandatory forecast baselines at matching issuance, valid times and regional support. An optional model conditioned on those future trajectories would be a separately labeled experiment.

Synthetic targets are future downscaled states constructed under the same historical state policy; they are not automatically the long-lead forecast from the origin's GFS cycle. Their later availability is permissible as training targets, never as origin-time inputs.

BRAMS-only initialization, ERA5 experiments and other sources are alternative experiments requiring source-specific validation. Neither documentation nor good synthetic-target performance establishes operational readiness.

## Future observations and energy application

GOES may later add recent cloud/convection information alongside atmospheric fields. It does not define a universal 0.5 km grid for surface wind, temperature and pressure. Evaluate incremental skill with and without satellite inputs before expanding the MVP.

A separate consumer project may estimate wind generation and extreme-weather/outage risk. Add turbine-height winds, gust/lightning targets and relevant generation/infrastructure/interruption records before claiming these outputs. Preserve forecast uncertainty and provenance through that application layer.

## Outputs and acceptance gates

Each forecast export contains issuance, valid times, leads, coordinates, channels, masks, model version and input provenance. Metric reports include sample counts, geographic weighting and baseline errors.

Proceed to a learned model only after data windows and baselines are reproducible. Claim improvements only on held-out periods with paired forecast origins and block-based uncertainty estimates. Proceed to an operational demo only after source availability and distribution shift have been evaluated.

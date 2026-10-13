# forecast-forge

Regional, multivariate weather forecasting for Northeast Brazil.

## Objective

Develop and evaluate forecasts up to 72 hours ahead on the regional grid used by cosmic-downscaler, with nominal 8 km resolution. Recent GFS fields transformed by cosmic-downscaler are the primary input. Train and operate with a versioned, availability-aware GFS cycle/lead policy. Operational readiness still requires validation of latency, coverage and forecast skill.

This repository learns temporal evolution. The companion cosmic-downscaler learns spatial refinement. A spatial downscaler alone does not predict future weather.

## MVP scope

| Item | Initial scope |
| --- | --- |
| Domain | Longitude −49° to −33°; latitude −19° to +1°, including ocean and a regional buffer |
| Grid | Same versioned BRAMS reference grid as cosmic-downscaler; nominal 8 km |
| Cadence | 6 hours, provisional MVP default |
| Forecast leads | +6, +12, …, +72 hours |
| Initial history | Proposed 24-hour window: t−24, t−18, t−12, t−6, t |
| Fields | 2 m temperature, 2 m dewpoint, 10 m zonal and meridional wind, mean sea-level pressure |
| Derived product | Wind speed calculated from predicted wind components |
| Baselines | Persistence and daily-cycle persistence |
| First learned model | Compact spatial-temporal model, architecture selected after data profiling |

The bounding box is a broad regional domain, not an administrative Northeast boundary. Preserve the ocean and buffer for modeling; use a separate geographic mask for regional reporting.

Precipitation, extra pressure levels, probabilistic ensembles, deployment and additional atmospheric variables are later extensions. Adding fields requires compatible reference data, units and evaluation.

## Data and companion project

cosmic-downscaler produces regional historical fields and associated provenance. forecast-forge consumes exported datasets through a versioned contract, rather than depending on another repository's internal modules.

Two experiments must remain distinct:

1. **Synthetic-field experiment:** predict future downscaled GFS states selected by the frozen state-construction policy. This measures agreement with the generated dataset.
2. **Reference evaluation:** compare forecasts with held-out BRAMS fields and, when available, compatible observations. BRAMS is a numerical-model reference, not observational truth.

Success on synthetic targets alone does not establish real-world forecasting skill. The initial model uses recent regional states through the forecast origin; future GFS forecast fields are excluded from its inputs. Compare against native and downscaled GFS trajectories issued with the same available information. Alternative sources require separate adaptation.

## Evaluation

- Chronological training, validation and test periods, with rolling forecast origins.
- No future targets crossing split boundaries.
- Normalization and learned preprocessing fitted only on training data.
- Frozen downscaler checkpoints trained without validation/test target information.
- MAE, RMSE and bias by field and lead; wind-vector error and derived speed error.
- Skill relative to both persistence baselines; geographic maps and land/ocean breakdowns.
- Explicit source, initialization, valid time, availability time and missing-data masks.

## Development

Use the existing Python 3.11 / uv project configuration. Install only dependencies required by implemented code.

```bash
uv sync --locked
uv run --locked ruff check .
uv run --locked pytest
```

Model training and forecasting commands will be documented when implemented.

## Milestones

1. Validate the companion data contract, five-field extraction and common grid.
2. Build a small continuous historical dataset and audit paired GFS/BRAMS availability.
3. Implement temporal windows, leakage checks and persistence baselines.
4. Train a compact model and compare on untouched historical periods.
5. Validate independent references and sensitivity to generated-field errors.
6. Validate the GFS publication policy in an operational replay and compare against its native and downscaled forecasts.

Current status: repository foundation and CI established; weather-modeling implementation pending.

See [architecture](docs/architecture.md) and [decisions](docs/decisions.md).

## Future application layer

GOES is a future complementary observation source for multimodal forecasting and convection experiments. Wind energy, extreme-weather guidance and power-outage risk belong to a separate application project. They require turbine-height winds and power curves, dedicated gust/lightning targets, and infrastructure/interruption data, respectively. The current five-field MVP does not provide those capabilities.

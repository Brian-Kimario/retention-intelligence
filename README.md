# Retention Intelligence Platform

A Python platform scaffold for customer-retention data strategy, prediction, decisioning, serving, and monitoring. The design distinguishes churn risk, customer value, treatment response, and business action; public predictive and uplift datasets are not combined.

## Project layout

- `docs/`: numbered business, data, temporal-design, target, policy, and dataset documents, plus the data contract, model card, and monitoring strategy.
- `data/`: immutable raw inputs, validated inputs, processed data, and external reference data. Existing silver and gold layer directories are retained under `data/processed/`.
- `notebooks/`: ordered acquisition, quality, exploration, feature, model, and decision-policy notebooks.
- `src/data/`: ingestion and validation code.
- `src/features/`, `src/models/`, `src/decision/`, and `src/monitoring/`: reusable platform logic.
- `src/serving/api/`: serving/API implementation location retained from the original scaffold.
- `models/`: model artifact storage.
- `tests/`, `configs/`, and `requirements/`: tests, runtime configuration, and dependency requirement files.
- `deployment/`: Docker, Kubernetes, Prometheus, and Grafana assets.
- `.github/workflows/`: CI workflow location.

## Getting started

1. Create and activate a Python virtual environment.
2. Install project dependencies from the chosen requirements file when one is defined.
3. Place unmodified source data in `data/raw/` and external reference data in `data/external/`.
4. Validate inputs before promoting them to `data/validated/`; store derived datasets in `data/processed/`.
5. Work through the notebooks in order, moving reusable logic into `src/`.

The scaffold intentionally does not invent a business schema or claim production performance from public data. See the numbered documents in `docs/` for the data, target, timing, and decision assumptions.

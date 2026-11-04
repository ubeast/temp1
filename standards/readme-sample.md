# shipment-analytics

Finds duplicate shipments and builds a daily carrier summary.

| | |
|---|---|
| **Owner** | Data Engineering – Logistics team (`#logistics-data` channel, logistics-data@example.com) |
| **Status** | In production |
| **GitLab group** | DATA-ENG / LOGISTICS |
| **Created** | 2026-11-04 |

## Overview

Carriers sometimes send the same shipment twice with slightly different details, which inflates volume and cost reports. This project finds those duplicates, keeps one clean record per shipment, and builds a daily summary of shipments, delays and cost per carrier. The summary feeds the Logistics Operations dashboard that planners use each morning.

## Data

| Direction | Name (`catalog.schema.table` or path) | Description |
|---|---|---|
| Input | `logistics_prod.raw.carrier_shipments` | Shipment records loaded hourly from carrier feeds. |
| Input | `logistics_prod.reference.carriers` | Carrier names, codes and contract rates. |
| Output | `logistics_prod.curated.shipments_deduplicated` | One row per shipment after duplicates are removed. |
| Output | `logistics_prod.curated.daily_carrier_summary` | One row per carrier per day: shipment count, average delay, total cost. |

The input contains customer delivery addresses, which are sensitive. The output tables drop the address columns, and access to `raw` is limited to the `logistics-engineers` group. Data refreshes daily at 05:00 UTC. The data owner is the Logistics Operations manager.

## Project structure

```text
shipment-analytics/
├── config/
│   ├── dev.yml
│   └── prod.yml
├── notebooks/
│   ├── exploration/
│   ├── 01_ingest_shipments.py
│   ├── 02_build_daily_summary.py
│   └── run_tests.py
├── src/
│   └── shipment_analytics/
├── tests/
│   └── test_transforms.py
├── .gitignore
├── AGENTS.md
├── pyproject.toml
└── README.md
```

The layout follows our file structure standard (in `AGENTS.md`).

## Requirements

| | |
|---|---|
| **Databricks Runtime** | 15.4 LTS |
| **Python** | >=3.11 |

Declared in `pyproject.toml`. Notebooks install them with a pinned `%pip install` in their first cell:

- `pyyaml==6.0.2`
- `requests==2.32.3`

You also need read access to `logistics_prod.raw` (ask to join the `logistics-engineers` group) and to the `logistics` secret scope.

## How to run

Run the notebooks in this order:

1. `notebooks/01_ingest_shipments.py` reads new rows from `raw.carrier_shipments`, removes duplicates and merges the result into `curated.shipments_deduplicated`.
2. `notebooks/02_build_daily_summary.py` rebuilds the previous day's rows in `curated.daily_carrier_summary`.

Both notebooks run as the scheduled job **shipment-analytics-daily** at 05:00 UTC, taking about 12 minutes in total. Both are safe to run again: notebook 1 uses `MERGE` and notebook 2 overwrites only the day it processes.

## Configuration

Settings live in `config/dev.yml` and `config/prod.yml`. Choose which one with the `environment` widget (default `dev`).

| Setting | What it controls |
|---|---|
| `catalog` | Unity Catalog catalog, e.g. `logistics_dev` or `logistics_prod`. |
| `run_date` | Widget: the day to process (`YYYY-MM-DD`). Defaults to yesterday. |
| `duplicate_window_hours` | How close two records' timestamps must be to count as duplicates (default 6). |

Secrets are read from Databricks secret scopes with `dbutils.secrets.get()`. Never put them in code or config.

| Secret scope | Key | Used for |
|---|---|---|
| `logistics` | `carrier-api-token` | Fetching late corrections from the carrier API in notebook 1. |

## Testing

Run `notebooks/run_tests.py` in Databricks. It runs every test in `tests/` and fails if any test fails.

Tests use small made-up DataFrames, so they never read the production tables.

## Contributing

1. Pull the latest `main`, then create a branch for your change.
2. Keep each commit to one logical change, with a message that says what changed and why.
3. Open a merge request in GitLab describing what changed, why, and how you tested it.
4. Get it reviewed before merging. Don't merge your own merge request without review.

See `CONTRIBUTING.md` if this project has one. Notable changes are recorded in `CHANGELOG.md`.

## Support

Ask questions in `#logistics-data`. Report bugs as GitLab issues on this project, with the run date and job run link.

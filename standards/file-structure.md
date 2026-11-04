<!--
  file-structure.md: the folder and file layout an AI assistant (Databricks Genie Code,
  Claude Code, or any other) must follow when it creates, moves or reviews files in our
  GitLab projects that are worked on in Databricks.

  For whoever maintains this next:
  - The three AI standards files (in docs/), each with one job:
      * naming-standard.md          WHAT to call things (this file only applies its rules)
      * file-structure.md           WHERE things go (this file)
      * engineering-standards.md    HOW to write code and docs
  - To take effect, run build_agents_md.py to rebuild AGENTS.md (Genie Code), or import the
    files from CLAUDE.md (Claude Code). See engineering-standards.md for the details.
  - Not decided yet, so marked "optional" below: Databricks Asset Bundles (databricks.yml and
    resources/), and whether notebooks are committed as source .py or as .ipynb. When decided,
    update section 2 and the tree in section 1.
-->

# File structure standard

Every GitLab project we work on in Databricks uses the layout below. Create new files in the folder this document names for them. If a file doesn't fit any folder here, ask before inventing a new top-level folder.

## 1. Standard project layout

```text
project-name/                      ← GitLab project (repo), kebab-case
├── README.md                      ← required: what, why, requirements, run, owner
├── pyproject.toml                 ← required for anything beyond a single script
├── .gitignore                     ← required
├── .gitlab-ci.yml                 ← CI/CD pipeline (when the project has one)
├── CHANGELOG.md                   ← when others rely on releases or scheduled jobs
├── CODEOWNERS                     ← when specific people must review specific paths
├── databricks.yml                 ← optional: Databricks Asset Bundle definition
│
├── src/
│   └── package_name/              ← snake_case; the importable Python package
│       ├── __init__.py
│       ├── config.py              ← loads settings from config/; no values hardcoded here
│       ├── io.py                  ← reading and writing tables and files
│       └── transforms.py          ← business logic, one module per concern
│
├── notebooks/                     ← Databricks notebooks: orchestration and exploration
│   ├── 01_ingest_shipments.py     ← source-format notebook: snake_case
│   ├── 02_build_daily_summary.py
│   ├── run_tests.py               ← runs pytest on tests/ (copy of the standard one)
│   └── exploration/               ← scratch analysis; not run by jobs
│       └── carrier-delay-review.ipynb   ← .ipynb notebook: kebab-case
│
├── tests/                         ← pytest tests, mirroring src/
│   ├── conftest.py
│   └── test_transforms.py
│
├── config/                        ← settings per environment; no secrets
│   ├── dev.yml
│   ├── test.yml
│   └── prod.yml
│
├── resources/                     ← optional: job and pipeline definitions for the bundle
│   └── daily-summary-job.yml
│
├── sql/                           ← stand-alone SQL (DDL, views, queries), kebab-case
│   └── create-shipment-events.sql
│
├── docs/                          ← longer documentation, kebab-case
│   ├── architecture.md
│   └── data-dictionary.md
│
└── scripts/                       ← helper scripts for developers or CI, snake_case .py
    └── refresh_test_data.py
```

Only `README.md` and `.gitignore` are required in every project. Add the other folders when the project needs them; don't create empty folders "just in case".

## 2. What goes where

| Folder / file | Put here | Don't put here |
|---|---|---|
| `src/package_name/` | Reusable logic: transforms, validation, I/O helpers, config loading. Everything that should be tested. | Notebook-only exploration, hardcoded settings, secrets. |
| `notebooks/` | Notebooks that jobs run, numbered in run order, plus a short first Markdown cell (purpose, inputs, outputs, how to run). Each notebook mostly calls functions from `src/`. | Large blocks of business logic copied between notebooks. Outputs with real data. |
| `notebooks/exploration/` | One-off analysis and experiments. | Anything a job or another notebook depends on. Move it to `notebooks/` or `src/` once it is relied on. |
| `tests/` | pytest files named `test_<module>.py`, one per `src/` module; small made-up test data. Run them with `notebooks/run_tests.py`. | Real or sensitive data. |
| `config/` | Non-secret settings per environment: catalog, schema and table names, paths, thresholds. | Tokens, passwords, keys (use Databricks secret scopes). |
| `resources/` | Optional. Job and pipeline YAML used by `databricks.yml`. | Settings that belong in `config/`. |
| `sql/` | DDL, view definitions and reusable queries. | SQL that only one notebook uses (keep it in that notebook). |
| `docs/` | Architecture notes, data dictionaries, runbooks, decision records. | The project overview (that is `README.md`). |
| `scripts/` | Developer and CI helpers that aren't part of the package. | Logic that jobs depend on (put it in `src/`). |
| Project root | Only the standard repo files in section 4 and the folders above. | Loose notebooks, scripts, data files or notes. |

## 3. Notebooks and `src/` in a Databricks Git folder

A Databricks Git folder is the workspace copy of the GitLab project, usually at `/Workspace/Users/<your-email>/<project-name>/`. To import our package from a notebook in `notebooks/`, add `src/` to the Python path in the first code cell:

```python
import sys
from pathlib import Path

# The notebook runs with its own folder as the working directory, so go up one level to the
# project root. This keeps the path relative, so it works in every user's Git folder.
SRC_DIR: Path = Path.cwd().parent / "src"
if str(SRC_DIR) not in sys.path:
    sys.path.append(str(SRC_DIR))

from package_name.transforms import deduplicate_shipments
```

- Use a relative path like this, never a hardcoded `/Workspace/Users/<someone>/…` path.
- A notebook in a subfolder (e.g. `notebooks/exploration/`) goes up one more level: `Path.cwd().parent.parent / "src"`.
- If the project is deployed as a wheel or bundle, jobs import the installed package instead and this cell isn't needed there.

## 4. Root files that keep their exact names

The special files listed in `naming-standard.md` go in the project root. Three need more:

- `README.md`: required in every project root. Always created from `readme-template.md`, by `create_readme.py` or by the AI. A subfolder may have its own.
- `AGENTS.md`: our standards for Genie Code, built by `build_agents_md.py`. Never edit it by hand.
- `LICENSE`: DoD code uses the J4LM license text.

## 5. What never goes in the repo

Make sure `.gitignore` covers at least:

```gitignore
# Python
__pycache__/
*.py[cod]
.venv/
.pytest_cache/
.ruff_cache/
dist/
build/
*.egg-info/

# Notebooks
.ipynb_checkpoints/

# Databricks
.databricks/

# Local settings and secrets
.env
*.env
.DS_Store
.vscode/
.idea/

# Data extracts
*.csv
*.parquet
*.xlsx
data/
```

If a small, non-sensitive sample file is genuinely needed for tests, put it under `tests/data/` and add an exception (`!tests/data/*.csv`) with a comment saying why.

## 6. GitLab group and project hierarchy

How group and project names are written is in `naming-standard.md`.

- Groups and subgroups organise teams; they hold projects, not files.
- One GitLab project per deployable product or pipeline. Don't put unrelated work in one project, and don't split one pipeline across several projects without a reason.

## 7. Rules for the AI when creating or moving files

- Before creating a file, decide its folder from section 2 and its name from `naming-standard.md`.
- Don't create new top-level folders, and don't move or rename existing files, unless asked. Moves break imports, notebook paths and job definitions; if a move seems worthwhile, suggest it and list what would need updating.
- When you see a project that doesn't follow this layout, point out the differences and suggest the smallest fix. Don't restructure it unasked.
- Keep the root clean: nothing goes there except the files in section 4 and the folders in section 1.

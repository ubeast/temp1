<!--
  naming-standard.md: the naming rules an AI assistant (Databricks Genie Code, Claude Code, or
  any other) must follow when it names or reviews GitLab groups, projects, folders, files,
  tables, columns and code in our GitLab projects that are worked on in Databricks.

  For whoever maintains this next:
  - Source of truth: lesson-04-file-naming/lessons/lesson-04-item-03-naming-standardization-guide.md
    (also as .html and .pdf). That guide is for people and explains WHY each rule exists. This
    file is the short AI version: rules and examples only. Change the guide first, then this.
  - The three AI standards files, each with one job:
      * naming-standard.md          WHAT to call things (this file)
      * file-structure.md           WHERE things go
      * engineering-standards.md    HOW to write code and docs
  - To take effect, run build_agents_md.py: it joins the three files (without these comments)
    into AGENTS.md, which goes in the root of each GitLab project. Never edit AGENTS.md by
    hand. Genie Code reads at most 20,000 characters; the build reports the size, about
    19,200 on 2026-11-04, so trim before adding rules.
  - Not decided yet, so not stated below: branch names (suggested <type>/<kebab-description>,
    e.g. feature/add-shipment-dedup) and secret scope / key names. Add them here once agreed.
-->

# Naming standard

Use these names for anything you create or rename. Before you suggest a name, check it against this file.

## 1. Case style by type

| What | Style | Example |
|---|---|---|
| GitLab groups and subgroups | ALL-CAPS | `DATA-ENG`, `TRAINING-DIV` |
| GitLab projects (repos) | kebab-case | `shipment-analytics` |
| Folders | kebab-case | `notebooks/`, `01-ingest/` |
| Markdown and other docs | kebab-case | `tech-setup.md` |
| Config and SQL files (YAML, JSON, TOML, SQL) | kebab-case | `dev-settings.yml`, `create-shipment-events.sql` |
| Notebooks saved as `.ipynb` | kebab-case | `find-duplicate-shipments.ipynb` |
| Python files, including notebooks in source format (`.py`) | snake_case | `data_loader.py`, `01_ingest_shipments.py` |
| Python package folders (under `src/`) | snake_case | `src/shipment_analytics/` |
| Unity Catalog catalogs, schemas, tables and columns | snake_case | `logistics_dev.curated.shipment_events`, `carrier_id` |
| Functions, variables | snake_case | `load_shipments()`, `run_date` |
| Classes | PascalCase | `ShipmentLoader` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Special files (section 2) | Exact conventional name | `README.md`, `AGENTS.md`, `.gitlab-ci.yml` |

Reading a full path:

```text
DATA-ENG / TRAINING-DIV / shipment-analytics / notebooks / 01_ingest_shipments.py
└──────── groups ──────┘ └──── project ────┘ └──────── folders and files ────────┘
     (ALL-CAPS)              (kebab-case)          (kebab-case / snake_case for .py)
```

## 2. Special files that keep their own case

Write these exactly as shown: `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `LICENSE`, `CODEOWNERS`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `Dockerfile`, `Makefile`, `.gitlab-ci.yml`, `.gitignore`, `pyproject.toml`, `AGENTS.md`, `CLAUDE.md`.

A file is special only if at least one of these is true:

1. A tool looks for it by that exact name (GitLab reads `CODEOWNERS`, Genie Code reads `AGENTS.md`).
2. The name is a convention almost every project uses (`README.md`, `CHANGELOG.md`).

Our own documents never qualify, however important they are: `coding-guide.md`, not `CODING-GUIDE.md`. Don't invent new ALL-CAPS names.

## 3. Rules

- No spaces and no capital letters in folder or file names, except the special files in section 2.
- No hyphens in Python file or package names: `import my-module` is a syntax error.
- A `.py` file in `notebooks/` that starts with `# Databricks notebook source` is a notebook. It is still a `.py` file, so it uses snake_case.
- Use a two-digit prefix (`01-`, `02-` for kebab-case; `01_`, `02_` for snake_case) only when files or folders must run or be read in order.
- Keep names short and descriptive: say what it is or does (`find-duplicate-shipments`), not who made it or when (`mikes-notebook-v2-final`). Don't put dates or version numbers in file names; Git keeps the history.

## 4. Checking a name

1. Is it one of the special files in section 2? Spell it exactly as listed.
2. Is it Python (`.py`, or a package folder)? Use snake_case.
3. Is it a GitLab group or subgroup? Use ALL-CAPS.
4. Anything else (project, folder, doc, config, `.ipynb`): use kebab-case.

When an existing name breaks the standard, point it out but don't rename it unless asked. Renames can break imports, links, jobs and other people's open branches. If a rename is worthwhile, list what would need updating.

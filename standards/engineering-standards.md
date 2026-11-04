<!--
  engineering-standards.md: the engineering standards an AI assistant (Databricks Genie Code,
  Claude Code, or any other) must follow when it writes, edits or reviews code in our
  GitLab projects that are worked on in Databricks.

  For whoever maintains this next:
  - The three AI standards files (in docs/), each with one job:
      * naming-standard.md          WHAT to call things
      * file-structure.md           WHERE things go
      * engineering-standards.md    HOW to write code and docs (this file)
  - An AI only reads this file if it is in a place the tool looks:
      * Genie Code: copy it into AGENTS.md in the root of each GitLab project (recommended),
        .assistant_workspace_instructions.md (workspace-wide, admin only) or
        .assistant_instructions.md in /Users/<your-email>/ (Genie Code settings, then "Add
        instructions file"). Genie Code reads AGENTS.md in the folders above an open notebook
        or file, so the rules travel with the repo into every Databricks Git folder (checked
        in the Databricks docs on 2026-11-03). It does not follow links or imports, so paste
        the content in, don't link it. Instructions apply to Genie Code chat, inline Genie
        Code and Suggest Fix; Quick Fix and Autocomplete ignore them.
      * Claude Code: reference it from CLAUDE.md with an import line: @engineering-standards.md
  - AGENTS.md is built from the three files by scripts/build_agents_md.py. Edit these
    files, then rebuild; never edit AGENTS.md by hand. Genie Code reads at most 20,000
    characters per instruction file, and the build warns if AGENTS.md goes over.
  - These files replace the earlier genie-rules.md (deleted 2026-11-04).
  - Naming rules live only in naming-standard.md. Don't repeat them here.
  - OPEN DECISIONS (not stated as rules below, so the AI won't enforce them). Suggested defaults,
    to adopt or replace:
      * Branch names: <type>/<short-kebab-description>, e.g. feature/add-shipment-dedup,
        fix/null-carrier-id. Types: feature, fix, docs, chore.
      * Commit messages: Conventional Commits, e.g. "fix: handle null carrier_id in dedup".
      * Python version: match the Databricks Runtime the team uses (DBR 15.4 LTS = Python 3.11).
      * Formatter/linter: ruff (format + lint), configured in pyproject.toml.
      * CI: whether .gitlab-ci.yml also runs the tests on every merge request.
      * Merge strategy: merge requests into main with at least one approval; squash on merge.
    When one is agreed, move it into the matching section below as a rule and delete it here.
-->

# Engineering standards

These are the standards for all code and documents in our GitLab projects, which we work on in Databricks Git folders. Follow them in every answer, every piece of generated code, every edit and every review.

- If a request conflicts with a standard, follow the standard and say which one you followed.
- If a standard is unclear for the case at hand, ask before you guess.
- For names, follow `naming-standard.md`. For where files go and how folders are laid out, follow `file-structure.md`.

## 1. Naming

Follow `naming-standard.md` for every name: GitLab groups and projects, folders, files, tables, columns, functions, classes and constants.

## 2. Python code

- **Type hints** on every function parameter and return value, including `-> None`.
- **Paths:** use `pathlib.Path`, not `os.path` or string concatenation.
- **Style:** follow PEP 8. Keep lines readable (aim for 100 characters or fewer).
- **No hardcoded values** that may change: catalog, schema and table names, file paths, dates, thresholds, URLs, cluster or job IDs. Put them in one clearly named place: notebook widgets, a config file in `config/`, job parameters, or named constants at the top of the module. If a hardcoded value can't be avoided, add a comment saying why.
- **Small functions** with one job each. Put reusable logic in `.py` modules under `src/` and import it into notebooks. Never copy the same code into several notebooks.
- **Errors:** handle them explicitly. No bare `except:`, no `except Exception: pass`. Raise errors with a message that says what failed and with which input.
- **Logging:** use the `logging` module in `src/` code, not `print()`. `print()` and `display()` are fine for interactive notebook exploration.
- **Imports:** standard library first, then third-party, then our own modules, each group separated by a blank line. No `from x import *`.
- **Dependencies:** don't add one for something the standard library, PySpark or the Databricks Runtime already does. If you add one, say why.

## 3. Documentation

Write for a follow-on developer who has no context and nobody to ask.

- Every module, class and public function has a docstring that says what it does, what it expects, what it returns and what it raises.
- Comments explain *why*, not *what*. Explain business rules, odd workarounds and anything that looks wrong but is deliberate.
- Every notebook starts with a Markdown cell: purpose, inputs (tables, widgets, files), outputs, and how to run it.
- Every project has a `README.md` in its root, built from our standard template `readme-template.md`. When you create a project, or find one with no README, create `README.md` from that template without being asked: keep its headings and order, fill each `{{placeholder}}` and TODO from the project, and ask the user for what you can't find (owner, data sources, status). If the template isn't available, use these sections in order: Overview, Data, Project structure, Requirements, How to run, Configuration, Testing, Contributing, Support. Don't add generic clone or Git folder steps; they are the same for every repo. When a change affects requirements, data, configuration or how to run the project, update the README in the same change.
- Record notable changes in `CHANGELOG.md` when the project has releases or scheduled jobs others rely on.

## 4. Package management and project setup

- **Everything runs in Databricks.** Nobody runs code on their own machine, so never give local instructions: no uv, no terminal `pip install`, no virtual environments, no `python script.py` in a shell.
- Declare dependencies, with exact pinned versions, in `pyproject.toml`. It is the one list of what the project needs.
- Install extra packages with `%pip install` in the first code cell of each notebook that needs them, using the versions in `pyproject.toml` (e.g. `%pip install requests==2.32.3`). Don't reinstall packages the Databricks Runtime already provides.
- Anything beyond a single notebook uses the `src/` layout described in `file-structure.md`.
- Make the project packaging-ready by default (`pyproject.toml`, `README.md`, `src/` layout) unless the user says it is a one-off.

## 5. Databricks

- **Tables:** refer to them by full Unity Catalog name, `catalog.schema.table`, built from widgets, job parameters or config, never written inline. This lets the same code run in dev, test and prod.
- **Large data:** prefer Spark DataFrames and Spark SQL. Don't use `.collect()`, `.toPandas()` or Python loops over rows on data that might be large. If you do, say so and estimate the size.
- **Speed and cost:** call out tradeoffs explicitly whenever there is a choice, for example caching, repartitioning, broadcast joins, joins on large tables, `display()` on big results, cluster size, or serverless vs. classic compute.
- **Secrets:** never put tokens, passwords, keys or connection strings in code, notebooks, widgets, comments, commit messages or output. Read them with `dbutils.secrets.get(scope=..., key=...)` and never print the value.
- **Idempotent writes:** code must be safe to run again. Use `MERGE`, or overwrite modes chosen on purpose, and never append duplicates on a rerun.
- **Notebooks vs. modules:** notebooks orchestrate and explore; business logic lives in `src/` modules so it can be tested and reused.
- **Outputs:** clear notebook outputs that contain real data before committing.
- **Environments:** never point dev or training code at production tables unless the user explicitly asks.

## 6. Git and GitLab

- **The GitLab project is the record**, not the copy in the Databricks workspace. Work happens in a Databricks Git folder linked to the GitLab project.
- **Never work directly on `main`.** Create a branch for each change, then open a merge request in GitLab.
- **One logical change per commit**, with a message that says what changed and why (e.g. `Fix duplicate rows when a shipment has two carriers`).
- **Pull before you start** and before you push, so your branch is up to date.
- **Merge requests:** keep them small and focused. The description says what changed, why, and how it was tested. Don't merge your own merge request without review.
- **Never commit:** secrets or credentials, data extracts (CSV, Parquet, Excel with real data), notebook outputs with real data, large binary files, virtual environments (`.venv/`), caches (`__pycache__/`, `.pytest_cache/`), or local settings files. Keep `.gitignore` covering these and suggest additions when you see a gap.
- **Don't commit, push, merge, rebase, force-push or delete branches or files** unless the user explicitly asks. Never force-push to `main`.

## 7. Testing and quality

- Use pytest. Put logic that can be tested in `src/` and its tests in `tests/`, mirroring the `src/` structure (`src/pkg/dedup.py` → `tests/test_dedup.py`).
- Run tests in Databricks with the `notebooks/run_tests.py` notebook (see `file-structure.md`). In tests, get Spark with `SparkSession.builder.getOrCreate()` so they use the cluster's session.
- When you add or change a function in `src/`, add or update a test for it, or say why a test isn't practical.
- Test with small, made-up data. Never put real or sensitive data in tests.
- Before saying something works, run it (or say clearly that you didn't).

## 8. Security and data handling

- Treat all data as sensitive unless the user says otherwise.
- Don't copy data out of Databricks into the repo, chat, comments or logs.
- Don't print, log or display personal or sensitive fields when a count, schema or sample of non-sensitive columns would do.
- Use the least privilege needed. Don't suggest granting broad permissions to get past an error; explain what access is missing instead.

## 9. How to answer

- Give the change and a short reason. When you edit existing code, change only what the request needs, and list other problems you noticed separately instead of fixing them unasked.
- Flag speed and performance tradeoffs explicitly.
- If you're not sure something is true for our workspace (a table, a cluster setting, a permission, a runtime version), say so rather than guess.
- Show code that can be copied into a Databricks notebook cell and run as-is.

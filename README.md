# Largest Banks ETL (Wikipedia → CSV + SQLite)

A production-ish ETL mini project that fetches the **LIVE** “By market capitalization” table from Wikipedia (*List of largest banks*), converts market cap from **USD** into **GBP / EUR / INR** using a local exchange rate CSV, and loads results to **CSV** and **SQLite**. Includes a CLI, Makefile shortcuts, Docker support, and CI smoke test.

## Features

* **Extract**: pulls the live Wikipedia page and parses the “By market capitalization” table
* **Transform**: converts `MC_USD_Billion` to `MC_GBP_Billion`, `MC_EUR_Billion`, `MC_INR_Billion` (rounded to 2 decimals)
* **Load**:

  * writes a CSV output
  * writes a SQLite database + table
* **Query**: supports running example SQL queries (or querying the DB yourself)
* **Logging**: writes an execution log file
* **DX**: Makefile targets + Docker image + CI (pytest + CLI smoke run)

---

## Project layout

```
.
├── src/                      # package source code (includes CLI)
├── data/
│   └── exchange_rates.csv    # USD -> GBP/EUR/INR rates
├── outputs/                  # generated outputs (gitignored)
├── pyproject.toml
├── Makefile
├── Dockerfile
├── .dockerignore
├── etl_project_log.txt       # generated log (gitignored)
└── README.md
```

---

## Quickstart (local)

### 1) Create venv + install

```python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -e .
```
Note: Change the python3 to your python version

### 2) Run the ETL pipeline (CLI)

```bash
make run
```

This will:

* create `outputs/`
* run the CLI with default arguments:

  * `--url "https://en.wikipedia.org/wiki/List_of_largest_banks"`
  * `--rates "data/exchange_rates.csv"`
  * `--out-csv "outputs/largest_banks.csv"`
  * `--db "outputs/largest_banks.db"`
  * `--table "Largest_banks"`
  * `--log "etl_project_log.txt"`

### 3) Run tests

```bash
make test
```

### 4) Clean generated artifacts

```bash
make clean
```

---

## CLI usage

Run directly (after installation):

```bash
largest-banks-etl \
  --url "https://en.wikipedia.org/wiki/List_of_largest_banks" \
  --rates "data/exchange_rates.csv" \
  --out-csv "outputs/largest_banks.csv" \
  --db "outputs/largest_banks.db" \
  --table "Largest_banks" \
  --log "etl_project_log.txt"
```

### Parameters

* `--url`: Wikipedia page URL (live source)
* `--rates`: path to exchange rate CSV
* `--out-csv`: output CSV path
* `--db`: SQLite database file path
* `--table`: SQLite table name
* `--log`: log file path

---

## Outputs

After a successful run:

### CSV

`outputs/largest_banks.csv`

Typical columns:

* `Name`
* `MC_USD_Billion`
* `MC_GBP_Billion`
* `MC_EUR_Billion`
* `MC_INR_Billion`


## Run with Docker

### Build

```bash
docker build -t largest-banks-etl .
```

### Run (write outputs to your local folder)

```bash
mkdir -p outputs
docker run --rm -v "$PWD/outputs:/app/outputs" largest-banks-etl
```

### Override args

```bash
docker run --rm -v "$PWD/outputs:/app/outputs" largest-banks-etl \
  largest-banks-etl \
  --url "https://en.wikipedia.org/wiki/List_of_largest_banks" \
  --rates "data/exchange_rates.csv"
```

---

## Development extras (optional but strong for GitHub)

### Pre-commit (ruff lint + format)

Install dev tools:

```bash
pip install -r requirements-dev.txt
pre-commit install
pre-commit run --all-files
```

Recommended `requirements-dev.txt`:

```txt
pytest
ruff
pre-commit
```

Recommended `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.4
    hooks:
      - id: ruff
        args: ["--fix"]
      - id: ruff-format
```

Recommended `pyproject.toml` additions:

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I"]
```

---

## CI (GitHub Actions)

This repo includes a CI workflow that runs:

1. `pytest`
2. A **CLI smoke test** that executes the full ETL run once (so you don’t end up with a “imports-only” project)

Workflow file:
`.github/workflows/ci.yml`

---

## Notes / Common issues

### Wikipedia table changes

Wikipedia HTML can change. If extraction breaks (e.g., empty dataframe / missing table), update the selector logic to match the current table structure.

### lxml / parsing dependencies

Dockerfile installs system libraries needed for `lxml` (commonly required by HTML table parsing).

---

## What data engineering skills are included in this project

This project demonstrates:

* external data ingestion (web)
* clean transformation logic and reproducible outputs
* relational persistence (SQLite) + SQL querying
* logging and basic pipeline ergonomics (CLI + Makefile + Docker + CI)

# Automated Job Postings Pipeline (v1) - dbt + DuckDB

![dbt pipeline diagram](img/dbt_pipeline.png)

A public dbt + DuckDB pipeline over ~692k real job postings, rebuilt and republished automatically on every push.

- Models the raw postings into tested, documented insight marts
- Publishes the docs site and the finished warehouse — open to anyone, no credentials
- Runs itself on GitHub Actions: every push and every Monday

> **v1 of this pipeline** - everything free and local-friendly: DuckDB as the warehouse, GitHub Actions as the scheduler. [v2](../project_2) rebuilds it production-grade on Databricks.

## What I Built

| What I Built          | How I Implemented                                                   | Where to Look                                                                                                       |
| --------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **dbt pipeline**      | Sources → cleaned models → analytical marts                         | [Analytics models](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/tree/main/analytics/models)                   |
| **Data tests**        | `unique` and `not_null` tests for data quality                      | [schema.yml](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/analytics/models/schema.yml)              |
| **dbt documentation** | Model descriptions, column documentation, and lineage configuration | [dbt project configuration](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/analytics/dbt_project.yml) |
| **GitHub Actions CI** | Automated workflow for building and testing the dbt project         | [dbt_build.yml](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/.github/workflows/dbt_build.yml)       |
| **Environments**      | Separate configuration for local development and CI                 | [CI configuration](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/tree/main/ci)                                 |
| **Pipeline scripts**  | Supporting scripts for the data pipeline                            | [Scripts directory](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/tree/main/scripts)                           |

## ⚙️ The dbt Pipeline

Raw job-posting data flows through the dbt transformation process into analytical models using DuckDB. The project includes cleaned job-posting records and analytical models for monthly trends, top locations, and remote-work metrics.

The main dbt models are:

- `job_postings` — cleaned job-posting records.
- `jobs_per_month` — monthly job-posting counts.
- `top_locations` — identifies the leading job-posting locations.
- `monthly_summary` — provides monthly summaries and remote-work metrics.

The dbt features doing the work:

| dbt Feature        | How I Used It                                                                      | Where to Look                                                                                               |
| ------------------ | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Model organization | Organized transformations into dbt models for job postings and analytical insights | [`analytics/models/`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/tree/main/analytics/models)        |
| Model dependencies | Used dbt model references to manage dependencies and execution order               | [`analytics/models/`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/tree/main/analytics/models)        |
| Materializations   | Configured model behavior through dbt project settings                             | [`dbt_project.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/analytics/dbt_project.yml) |
| Data tests         | Applied `unique` and `not_null` tests to validate data quality                     | [`schema.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/analytics/models/schema.yml)    |
| Environments       | Configured a production DuckDB target for pipeline execution                       | [`ci/profiles.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/ci/profiles.yml)           |

`dbt build` compiles SQL, builds the configured models, and executes associated data tests. A failed test causes the relevant build to fail.

---

## 🚀 Production

GitHub Actions provides workflow automation for the dbt pipeline, allowing the project to run through CI/CD rather than relying exclusively on a local machine.

**Workflow:** [`.github/workflows/dbt_build.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/.github/workflows/dbt_build.yml)

| Trigger                      | What Happens                                              |
| ---------------------------- | --------------------------------------------------------- |
| Push to `main`               | Intended pipeline rebuild                                 |
| Weekly schedule              | Scheduled pipeline execution                              |
| Manual (`workflow_dispatch`) | Allows manual execution from the GitHub Actions interface |

### What happens during a workflow run?

1. Checks out the GitHub repository.
2. Sets up the Python environment and installs dependencies using `uv`.
3. Downloads the raw job-posting data.
4. Runs `dbt build` using the configured CI production profile.
5. Attempts to publish the generated `prod.duckdb` warehouse as a GitHub Release asset.

**View the workflow:** [GitHub Actions](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/actions)

Note: The workflow currently needs its push trigger changed from `master` to `main` to match the repository's default branch. Successful warehouse publication should also be verified in GitHub Releases.

---

## 🦆 Query the DuckDB Warehouse

The project uses DuckDB as its analytical warehouse.

You can install DuckDB using the official instructions:

[DuckDB installation guide](https://duckdb.org/docs/installation/)

On macOS, you can install it with Homebrew:

```bash
brew install duckdb
```

After building the project locally, open the generated warehouse:

```bash
duckdb data/prod.duckdb
```

From the DuckDB session, inspect the available tables:

```sql
SHOW TABLES;
```

Query the analytical models:

```sql
SELECT * FROM top_locations;

SELECT * FROM monthly_summary;

SELECT * FROM jobs_per_month;

SELECT * FROM job_postings LIMIT 10;
```

The production database is configured through [`ci/profiles.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/ci/profiles.yml).

A public `ATTACH` example and downloadable warehouse URL can be added once the database has been successfully published as a GitHub Release asset.

---

## 📖 Docs

The project uses dbt's documentation functionality to generate documentation for models, columns, and data tests.

Documentation metadata is maintained in:

[`analytics/models/schema.yml`](https://github.com/Smartlyfe21/dbt-duckdb-pipeline/blob/main/analytics/models/schema.yml)

Generate the documentation locally:

```bash
uv run dbt docs generate --profiles-dir ../ci
```

Serve the generated documentation:

```bash
uv run dbt docs serve --profiles-dir ../ci
```

The generated documentation allows you to explore:

- **Model lineage:** relationships and dependencies between dbt models.
- **Model descriptions:** documentation maintained in YAML.
- **Column descriptions:** explanations of individual model columns.
- **Data tests:** configured quality checks on model columns.

The documentation can be published to GitHub Pages in a future deployment step.

---

## 💻 Run It Yourself

Clone the repository:

```bash
git clone https://github.com/Smartlyfe21/dbt-duckdb-pipeline.git

cd dbt-duckdb-pipeline
```

Install project dependencies:

```bash
uv sync
```

Download the raw job-posting data:

```bash
uv run python scripts/download_data.py
```

Navigate to the dbt project:

```bash
cd analytics
```

Build the pipeline:

```bash
uv run dbt build --profiles-dir ../ci
```

Optional: generate and serve dbt documentation:

```bash
uv run dbt docs generate --profiles-dir ../ci

uv run dbt docs serve --profiles-dir ../ci
```

---

## 📁 Project Layout

```text
dbt-duckdb-pipeline/
│
├── .github/
│   └── workflows/
│       └── dbt_build.yml       # GitHub Actions workflow
│
├── analytics/
│   ├── dbt_project.yml         # dbt project configuration
│   ├── models/                 # dbt models and schema definitions
│   │   └── schema.yml          # Model documentation and data tests
│   ├── macros/
│   └── seeds/
│
├── ci/
│   └── profiles.yml            # Production DuckDB target
│
├── scripts/
│   └── download_data.py        # Downloads raw job-posting data
│
├── src/
│   └── dbt_duckdb_pipeline/
│
├── pyproject.toml              # Python dependencies
├── uv.lock                     # Dependency lock file
└── README.md
```

---

_An analytics engineering project demonstrating data transformation, data quality testing, analytical modeling, DuckDB warehousing, and CI/CD automation using dbt and Python._

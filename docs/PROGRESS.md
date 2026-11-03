# Progress — Trailhead Supply Co. Data Platform

## Learner profile (set 2026-10-27)
| Tool | Level |
|---|---|
| SQL | comfortable |
| Python | comfortable |
| Snowflake | comfortable |
| Docker | none |
| Git | none (confirm) |
| AWS | none |
| Terraform | none |
| Airflow | none |
| dbt | none |

**Time budget:** a few hours every day.
**Order:** build on DuckDB first, then move to Snowflake (see ADR-002).

## Snowflake trial
- Status: **not started** (start at Phase 3, once Phases 4 and 6 work on DuckDB)
- Start date: —
- End date (start + 30 days): —

## AWS account (ADR-003)
- Name: `trailhead-data-platform`, Free plan
- Created: 2026-11-02
- Free plan ends: ~2027-05-02 (or when credits run out)
- Region: `us-east-2` (US East, Ohio) — CLI set 2026-11-02; confirm it matches AWS Settings → Project → Region
- CLI sign-in: `aws login --remote` (short-lived credentials, up to 12 hours)
- Budget: `trailhead-monthly-1usd` — $1/month, credits excluded, email alert at 80% actual (created by hand; rebuild in Terraform in Phase 2)
- Credits: balance/expiry not yet recorded
- MFA: skipped by choice (can add at settings.aws.com)

## Current position
- Phase 0, Lesson 0.4 — next (Git and repo setup)

---

## Lessons

Status: `todo` · `in progress` · `done` · `skipped`

Phases are listed in the order we'll do them (DuckDB first, then Snowflake).

### Phase 0 — Orientation and Setup
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 0.1 | The big picture: how a data platform fits together | done | 2026-10-27 | ELT keeps raw data so history can be rebuilt; Airflow coordinates, it doesn't process; S3 decouples extract from load |
| 0.2 | Install and verify tools | done | 2026-10-27 | Homebrew taps, PATH and why the shell can't find a command; system Python 3.9 vs project Python 3.12 via uv |
| 0.3 | AWS account on the Free plan, set up safely (MFA, IAM, $1 budget) | done | 2026-11-02 | Old account is legacy paid billing; Free plan can't charge; storage costs money even when nothing runs; new AWS accounts have no root user, a fixed region, and use `aws login` for short-lived CLI credentials; budgets must exclude credits to catch usage |
| 0.4 | Git basics, repo init, .gitignore, .env.example, pre-commit, first push | todo | | |

### Phase 0.5 — Docker Fundamentals
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 0.5.1 | Containers vs VMs, images, run/ps/logs/exec/stop/rm | todo | | |
| 0.5.2 | Volumes: losing data, then keeping it | todo | | |
| 0.5.3 | Networking: localhost inside containers, service names, ports | todo | | |
| 0.5.4 | Writing a Dockerfile (slim base, layer order, non-root user) | todo | | |
| 0.5.5 | Docker Compose: services, healthchecks, env files, profiles | todo | | |
| 0.5.6 | Housekeeping: image size, prune, tagging | todo | | |
| 0.5.M | Milestone: Postgres + Python container talking by service name | todo | | |
| 0.5.D | Failure drills: localhost trap, cache-busting Dockerfile | todo | | |

### Phase 1 — The Source System
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 1.1 | OLTP vs OLAP, designing a 3NF schema | todo | | |
| 1.2 | Postgres in Compose, DDL for the orders app | todo | | |
| 1.3 | Generator: seed 12 months of history | todo | | |
| 1.4 | Generator live mode: inserts, updates, soft deletes, late payments | todo | | |
| 1.5 | Clickstream JSON and ad-spend CSV files, with messiness | todo | | |
| 1.6 | Containerize the generator | todo | | |
| 1.D | Failure drill: malformed CSV + validation check | todo | | |

### Phase 2 — Terraform Foundations (AWS)
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 2.1 | Terraform core: providers, resources, state, plan/apply/destroy | todo | | |
| 2.2 | Bootstrap remote state in S3 with locking | todo | | |
| 2.3 | Raw S3 bucket: encryption, public access block, lifecycle | todo | | |
| 2.4 | IAM roles and least-privilege policies | todo | | |
| 2.5 | SSM Parameter Store and AWS Budget in Terraform | todo | | |
| 2.6 | Modules and environments (dev/prod) | todo | | |
| 2.D | Failure drill: console drift, detect and fix with plan | todo | | |

### Phase 4 — Ingestion with Airflow
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 4.1 | Airflow concepts: DAGs, tasks, data intervals, catchup | todo | | |
| 4.2 | Airflow in Docker + custom Airflow image | todo | | |
| 4.3 | DAG 1: incremental Postgres → Parquet → S3 | todo | | |
| 4.4 | DAG 2: FX rates API with retries and backoff | todo | | |
| 4.5 | DAG 3: pick up ad-spend files | todo | | |
| 4.D | Failure drill: run a day twice, find duplicates, make it idempotent | todo | | |

### Phase 6a — dbt on DuckDB (dev target)
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 6.1 | dbt project setup, profiles, DuckDB `dev` target | todo | | |
| 6.2 | Sources, staging models, freshness | todo | | |
| 6.3 | Snapshots (SCD2) for customers and products | todo | | |
| 6.4 | Intermediate: sessionize clickstream, orders + payments + FX | todo | | |
| 6.5 | Marts: star schema (facts and dims) | todo | | |
| 6.6 | Incremental models and late-arriving data | todo | | |
| 6.7 | Tests, packages and model contracts | todo | | |
| 6.8 | Docs site and exposures | todo | | |
| 6.9 | Run dbt in a container | todo | | |
| 6.D | Failure drill: duplicate order breaks a test | todo | | |

### Phase 3 — Terraform for Snowflake (start the trial here)
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 3.1 | Start the trial (AWS, Standard edition, same region as S3) | todo | | |
| 3.2 | Snowflake provider: databases, schemas, warehouses, resource monitor | todo | | |
| 3.3 | Roles, service users, key-pair auth | todo | | |
| 3.4 | Storage integration + IAM trust (wiring AWS ↔ Snowflake outputs) | todo | | |
| 3.5 | External stage and file formats; milestone `LIST @raw_stage` | todo | | |

### Phase 5 — Loading into Snowflake
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 5.1 | COPY INTO and load metadata | todo | | |
| 5.2 | VARIANT for JSON clickstream, schema drift | todo | | |
| 5.3 | Snowpipe auto-ingest via S3 → SQS | todo | | |
| 5.4 | Metadata columns (`_loaded_at`, `_source_file`) | todo | | |
| 5.D | Failure drill: bad row, compare ON_ERROR options | todo | | |

### Phase 6b — dbt on Snowflake (prod target)
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 6.10 | Add `prod` target, adapter differences, cross-database macros | todo | | |

### Phase 7 — End-to-End Orchestration
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 7.1 | dbt in Airflow with Cosmos | todo | | |
| 7.2 | Asset-driven scheduling | todo | | |
| 7.3 | Alerts, SLAs, retries | todo | | |
| 7.4 | Backfill a past month end-to-end | todo | | |

### Phase 8 — Data Quality and Observability
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 8.1 | Layered quality checks and row-count reconciliation | todo | | |
| 8.2 | Snowflake query history and cost per model | todo | | |
| 8.D | Failure drill: generator silently stops, alert must fire | todo | | |

### Phase 9 — CI/CD
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 9.1 | PR workflow: ruff, SQLFluff, pytest, DAG import tests | todo | | |
| 9.2 | Terraform plan on PR, apply on merge, AWS via OIDC | todo | | |
| 9.3 | dbt slim CI with `state:modified+` and `--defer` | todo | | |
| 9.4 | Building images in Actions, multi-stage builds | todo | | |

### Phase 10 — Serverless on AWS
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 10.1 | How companies host Airflow (MWAA vs ECS vs K8s) | todo | | |
| 10.2 | Lambda CSV validator with quarantine prefix + webhook alert | todo | | |

### Phase 11 — Serve the Data
| Lesson | Title | Status | Date | What I learned |
|---|---|---|---|---|
| 11.1 | Streamlit (or Metabase) dashboard on marts, REPORTER role | todo | | |
| 11.2 | Final README: diagram, setup, screenshots, what's next | todo | | |

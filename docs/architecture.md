# Architecture — Trailhead Supply Co. Data Platform

## Life of one order

A customer buys a tent on the website. This is how that order becomes part of the revenue number leadership sees.

1. **Postgres (source app database).** The app inserts a row into `orders` and one row per product into `order_items`. The payment may arrive minutes or hours later in `payments`. Postgres is built for these many small writes. We never run analytics on it.
2. **Airflow (extract).** Every night, an Airflow DAG asks Postgres for rows whose `updated_at` falls inside that run's time window (for example 2026-10-27 00:00 → 2026-10-28 00:00). That is an incremental extract: only new or changed rows, not the whole table. Airflow coordinates the job. It doesn't do heavy processing.
3. **S3 (landing zone).** The extracted rows are written as **Parquet** files, partitioned by date: `s3://<project>-raw/postgres/orders/dt=2026-10-27/`. Parquet is columnar, typed and compressed. Clickstream events land here as **JSON** and ad spend as **CSV**. S3 keeps a cheap history we can replay.
4. **Raw layer (bronze), in DuckDB or Snowflake.** The files are loaded into the warehouse exactly as they are, with no fixes. We keep this untouched copy so any later step can be rebuilt without going back to Postgres.
5. **Staging layer (silver), built by dbt.** One model per source table (`stg_orders`, `stg_payments`): rename columns, cast types, remove duplicates. Still 1:1 with the source tables.
6. **Intermediate layer (silver), built by dbt.** Business logic: join the order with its items and payments, and convert the amount to one currency using the FX rates.
7. **Marts layer (gold), built by dbt.** Star schema (`fct_orders`, `dim_customers`, `dim_products`, `dim_stores`, `dim_date`) and business marts. `mart_daily_revenue` adds the tent to that day's revenue by channel, store and category. The dashboard reads from here.

## Components

| Component | Its one job | What goes wrong without it |
|---|---|---|
| **Postgres** | Runs the business: stores orders, customers and inventory as they happen (OLTP, row store) | Nothing to analyze. If analysts queried it directly, heavy reports would slow down the live website |
| **S3** | Cheap, durable landing zone for raw files, separate from the warehouse | Extract and load are tied together: if the warehouse is down, extraction fails too, and there's no raw history to replay after a bug |
| **DuckDB / Snowflake** | Warehouse: stores raw and modeled data and runs analytical SQL fast (OLAP, column store) | Analytics on a row store is slow, and there's no single place for the "one source of truth" |
| **dbt** | Turns raw tables into tested, documented models with SQL kept in Git | Transforms live in scattered scripts and spreadsheets with no tests, no lineage and no review. That's how teams end up with three different revenue numbers |
| **Airflow** | Schedules and coordinates every step, in order, with retries and alerts | Jobs run by cron or by hand. Nobody knows what failed, what ran out of order, or which days are missing |
| **Terraform** | Defines S3 buckets, IAM, Snowflake roles and warehouses as code | Infrastructure gets built by clicking in consoles. It can't be reviewed, reproduced or torn down cleanly, and it drifts over time |
| **Docker** | Packages Postgres, Airflow and dbt with exact versions so they run the same everywhere | "Works on my machine" problems: different versions on your laptop, in CI and in production |

## Two "incrementals" (don't mix them up)

- **Incremental extract (Airflow):** pull only new or changed rows from the source for one time window.
- **Incremental model (dbt):** inside the warehouse, process only new rows instead of rebuilding the whole table.

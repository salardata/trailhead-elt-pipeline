# Architecture Decision Log

Short records of important choices: what we decided, what else we considered, and why.

---

## ADR-001 — Database roles
**Date:** 2026-10-27 · **Status:** accepted

**Decision**
- **PostgreSQL** is the source system. It plays the company's app database (OLTP) and runs in Docker. It is never used for analytics.
- **DuckDB** is the free local warehouse and the dbt `dev` target.
- **Snowflake** is the production warehouse during the 30-day trial.

**Alternatives considered**
- Postgres as both source and warehouse: one less tool, but it is a row store built for many small writes. Analytical queries (scan millions of rows, read few columns) run slowly, and it doesn't teach warehouse skills.
- Snowflake only: no free permanent dev target; the 30-day trial would run out while learning basics.

**Why**
Row stores fit transactions, column stores fit analytics. DuckDB gives a free, permanent column-store target. Running dev and prod targets in one dbt project is also a real-world habit.

---

## ADR-002 — DuckDB first, Snowflake later
**Date:** 2026-10-27 · **Status:** accepted

**Decision**
Build Phases 0, 0.5, 1, 2, 4 and 6 against DuckDB. Start the Snowflake trial at Phase 3, then do Phases 3, 5, 6 (prod target), 7 and 8 on Snowflake.

**Alternatives considered**
- Start the Snowflake trial now: trial days would be spent on Docker, Terraform and Airflow basics that don't need Snowflake.

**Why**
The trial lasts only 30 days. The learner already knows Snowflake, so the trial is best spent on Snowflake-specific pieces (storage integration, Snowpipe, RBAC in Terraform). With a few hours a day, 30 days is enough for those phases.

---

## ADR-003 — Use a fresh AWS account on the Free plan
**Date:** 2026-11-02 · **Status:** accepted

**Decision**
Create a new AWS account named `trailhead-data-platform` on the **Free plan**. Don't use the existing personal account for this project.

**Update 2026-11-02:** the account was created on 2026-11-02 (earlier than planned). The Free plan ends on about 2027-05-02, or sooner if the credits run out.

**Context**
The existing account was created before July 15, 2025. It uses the older paid billing model, its 12-month free tier has ended, and it isn't eligible for the new Free plan credits. Any usage above Always Free limits is charged to the card on file. An old Redshift Serverless namespace in us-east-2 was found there and deleted on 2026-11-02.

**Account type (found 2026-11-02):** the new account uses AWS's newer sign-up experience (rolled out from Sept 2026):
- There is **no root user**. Sign-in is with an AWS Builder ID (or Google/Apple/GitHub/Amazon). MFA is set at settings.aws.com → Manage security and sign-in.
- Work is organized into **projects**. People are invited to projects by email instead of being created as IAM users.
- IAM users and roles work only for programmatic access. **IAM Identity Center is not available.**
- S3, IAM, Lambda, SQS, Systems Manager and Budgets are all available.
- **Region is fixed:** AWS assigns one region from the contact-address country (us-east-2, eu-north-1 or ap-southeast-2), and it can't be changed. Cross-region features aren't supported.
- **CLI sign-in:** plan to use `aws login` (CLI 2.32+), which gives short-lived credentials that refresh for up to 12 hours. AWS docs don't yet confirm it for this account type; test it in Lesson 0.3 Part B.
- **Never activate "advanced features":** it can't be undone, requires upgrading the account, and removes spend limits.

**Alternatives considered**
- Keep the existing account and stay within Always Free limits with budget alarms: possible, but one forgotten resource means a real charge.

**Why**
The Free plan can't charge at all and comes with $100–$200 in credits as a buffer. Creating it right before Phase 2 means the 6-month window covers the AWS-heavy phases.

**Follow-ups**
- Old account: decide later whether to close it or add a $1 budget alarm.
- Check whether Glue or any Redshift snapshot is still on the old account's next bill.
- Phase 2: check that Terraform's AWS provider can use `aws login` credentials (fallback: `credential_process` with `aws configure export-credentials`).
- Phase 3 risk: AWS docs don't say whether IAM roles that trust an outside AWS account (needed for Snowflake's storage integration) are allowed. Test this early in Phase 3, and confirm Snowflake offers a trial in the project's region.

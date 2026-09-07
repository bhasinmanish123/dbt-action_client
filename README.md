# dbt-action_client

Central CI/CD logic for dbt + Snowflake (WIF OIDC) — Helia-style two-repo pattern.

## What's Here

```
.github/
  workflows/
    dbt-reusable.yml              ← reusable workflow (lint + build logic)
  actions/
    setup-snowflake/
      action.yml                  ← composite action (WIF OIDC token + connection)
```

## How Project Repos Use It

A project repo (e.g. dbt_snowflake_wif_client) calls the reusable workflow:

```yaml
# project-repo/.github/workflows/dbt_cicd.yml
jobs:
  ci:
    if: github.event_name == 'pull_request'
    uses: bhasinmanish123/dbt-action_client/.github/workflows/dbt-reusable.yml@main
    with:
      stage: ci
    secrets:
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
  cd:
    if: github.event_name == 'push'
    uses: bhasinmanish123/dbt-action_client/.github/workflows/dbt-reusable.yml@main
    with:
      stage: cd
    secrets:
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
```

## The Flow

```
project repo caller (dbt_cicd.yml)
    │  uses dbt-reusable.yml@main
    ▼
dbt-reusable.yml (this repo)
    │  uses ./.github/actions/setup-snowflake
    ▼
setup-snowflake/action.yml (this repo)
    │  fetches OIDC token, sets Snowflake env vars
    ▼
Snowflake WIF auth
```

## Auth

- WIF (OIDC), no passwords, no keys
- Only secret needed: SNOWFLAKE_ACCOUNT
- Snowflake service user with WORKLOAD_IDENTITY, subject = repo:...:environment:prod

## Benefits of the Two-Repo Pattern

- CI/CD logic maintained in ONE place (this repo)
- Many project repos reuse it
- Change logic once → all projects get the update

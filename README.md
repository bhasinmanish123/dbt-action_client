# dbt-action_client

Central **Snowflake WIF token provider** for dbt projects.

## What's Here (token only)

```
.github/actions/setup-snowflake/action.yml   ← composite action: WIF OIDC token
```

This repo does ONE thing: provide the Snowflake WIF (OIDC) token + connection env vars.
It does NOT run dbt. Each project repo owns its own dbt commands (inline).

## How Project Repos Use It

```yaml
# project-repo/.github/workflows/dbt_ci.yml (or dbt_cd.yml)
steps:
  - uses: actions/checkout@v4

  - name: Setup Snowflake (WIF token)
    uses: bhasinmanish123/dbt-action_client/.github/actions/setup-snowflake@main
    with:
      environment: ci          # or prod
      snowflake-account: ${{ secrets.SNOWFLAKE_ACCOUNT }}

  # ---- dbt commands are INLINE in the project repo ----
  - run: pip install dbt-snowflake
  - run: dbt deps || true
  - run: dbt build --target ci
```

## Architecture

```
dbt-action_client (this repo — SHARED, token only)
    setup-snowflake/action.yml → WIF OIDC token + env vars
         ▲
         │ uses
         │
project repos (own their dbt logic INLINE)
    dbt_ci.yml → token + lint + dbt build ci
    dbt_cd.yml → token + dbt build prod
```

## Why This Split

- Shared piece = ONLY auth (stable, rarely changes)
- dbt commands = in each project repo (full control per repo)
- Easy per-repo customization (seed, tags, full-refresh)
- No dependency on a shared workflow's dbt logic

## What the Composite Action Does

1. Install Snowflake CLI
2. Fetch WIF OIDC token (github-script)
3. Set SNOWFLAKE_TOKEN, AUTHENTICATOR, PROVIDER, USER, ROLE, DATABASE, WAREHOUSE
4. Test connection

## Auth

- WIF (OIDC), no passwords, no keys
- Only secret needed: SNOWFLAKE_ACCOUNT
- Snowflake service user with WORKLOAD_IDENTITY (subject = repo:...:environment:prod)

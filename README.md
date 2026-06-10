# Quality Rule Automation Shared

Shared quality-rule automation project extracted from the PH/TH intelligent alarm repair assistants.

This project keeps the existing behavior of the current automation flow, but moves the reusable parts into one standalone codebase so multiple country repos can depend on the same logic.

## Included workflows

- Scan tables that still need rule generation
- Generate a single table's candidate rule
- Submit candidates into a Google Form / Google Sheet confirmation flow
- Apply confirmed rules back into `wattrel`
- Optionally delete processed rows from the confirmation sheet
- Send batch summary notifications

## Directory layout

- `alert/`: DB connection helpers
- `config/`: runtime configuration
- `core/`: shared business logic
- `tools/`: CLI entrypoints
- `tests/`: behavior checks copied from the working PH flow

## Runtime model

All runtime values are environment-driven.

Nothing in this repository should require country-specific code edits. PH, TH, and future countries should only need different environment variables.

## Quick start

1. Create a local env file:

```bash
cp .env.example .env.local
```

2. Fill the required values in `.env.local`.

3. Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

4. Run tests:

```bash
python3 -m unittest discover -s tests -p '*checks.py'
```

## Main commands

List pending tables:

```bash
python3 tools/list_pending_quality_rule_tables.py \
  --database dwd \
  --database dim \
  --database dwd_sec \
  --json
```

Generate one table:

```bash
python3 tools/run_single_quality_rule_flow.py \
  --database dwd \
  --tbl dwd_user_activity_log
```

Apply confirmed rows:

```bash
python3 core/apply_confirmed_quality_rules.py \
  --export-url "$QUALITY_RULE_CONFIRMATION_EXPORT_URL" \
  --country ph \
  --json
```

Apply confirmed rows from n8n Google Sheets connector payload:

```bash
python3 core/apply_confirmed_quality_rules.py \
  --decision-json-base64 "$DECISION_ROWS_BASE64" \
  --country ph \
  --json
```

## Integration recommendation

For PH / TH / other country repos, prefer:

1. Keep country repo only for deployment wiring and env values
2. Consume this shared project as a git submodule, subtree, or synced directory
3. Pass country-specific runtime values through environment variables

## Notes

- By default, confirmed-rule apply does **not** do syntax validation unless `--validate-syntax` is passed.
- If Google service account credentials are configured, processed confirmation rows can be deleted automatically after successful apply.
- If credentials are not configured, the summary message will prompt manual row cleanup instead.

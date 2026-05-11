---
name: octoarms-x-collector
description: Use when you need a standardized way to query HBM list tweets from Octoarms contents API, verify recent collection results, and troubleshoot empty results.
---

# Query HBM Tweets

## Overview
Use this skill to query HBM list tweets through the Octoarms contents API in a consistent, repeatable way.

Target list for this skill is fixed:
- `list_id`: `2053711728324874698`

## When to Use
- You need to check whether HBM list data is being collected.
- You need a quick JSON proof for one or more tracked accounts.
- You need to troubleshoot why a recent run appears to have no data.

## Prerequisites
- `OCTOARMS_BASE_URL` is set.
- `OCTOARMS_API_KEY` is set and valid.
- HBM list member usernames are known (or fetched from source subscriptions).

```bash
export OCTOARMS_BASE_URL="https://api.chainbot.io"
export OCTOARMS_API_KEY="<YOUR_API_KEY>"
```

## API and Parameters

Endpoint:
- `GET /api/v1/scanner/contents`

Supported query parameters (current backend):

| Parameter | Type | Example | Meaning |
|---|---|---|---|
| `type` | string | `tweet` | Content type filter. For this skill, always use `tweet`. Default: no type filter. |
| `source` | string | `twitter` | Upstream source provider filter. For this skill, use `twitter`. Default: no source filter. |
| `keyword` | string | `HBM` | Full-text keyword search in content fields. Default: empty (disabled). |
| `username` | string | `jukan05` | Exact author username filter (no leading `@`). Default: empty (disabled). |
| `cursor` | string | `<opaque_cursor>` | Cursor for pagination (use response cursor if present). Default: first page. |
| `order_by` | string | `published_at_desc` | Sort order. Recommend `published_at_desc` for newest first. Default: backend default order. |
| `tags_mode` | string | `any` / `all` | How `tags` are combined. Default: backend default (typically `any`). |
| `source_tags_mode` | string | `any` / `all` | How `source_tags` are combined. Default: backend default (typically `any`). |
| `tags` | comma list | `macro,ai` | Content-level tag filtering. Default: empty (disabled). |
| `source_tags` | comma list | `hbm,tier_s` | Source-subscription tag filtering. Default: empty (disabled). |
| `limit` | int | `100` | Page size. Use moderate values (20/50/100). Default: backend default (commonly `100`). |
| `recent` | int (seconds) | `86400` | Relative time window from now. Default: disabled. |
| `published_after` | RFC3339 | `2026-05-10T00:00:00Z` | Absolute lower bound. Default: disabled. |
| `published_before` | RFC3339 | `2026-05-12T00:00:00Z` | Absolute upper bound. Default: disabled. |

Notes:
- `recent` and `published_after/published_before` can both be used; prefer one style per query for clarity.
- `username` should not include `@`.
- `source_name` and `source_id` are response fields, not guaranteed query filters in this API.

## Recommended Defaults
- `type=tweet`
- `source=twitter`
- `recent=86400` (last 24h)
- `limit=100`
- `order_by=published_at_desc`
- `username=<one list member username>`

## Standard Query (Single Account)

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=jukan05&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Expected signal in response items:
- `success: true`
- `source_name: "octoarms_hbm"`
- `username` matches query username

## How to Restrict to List `2053711728324874698`

Current practical restriction strategy:
1. Query only usernames that are members of list `2053711728324874698`.
2. Validate result rows contain `source_name: "octoarms_hbm"`.

Reason:
- Contents API currently does not expose a stable, explicit `list_id` filter in query params.
- Therefore list scoping is done by account scope + response verification.

### Step A: Get usernames for list `2053711728324874698`

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner-inter/sources/42/subscriptions?provider=twitter&limit=500" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Read usernames from `data.items[].meta_json.username`.

### Step B: Query by username and assert `source_name`

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=dnystedt&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Check each returned row:
- `source_name == "octoarms_hbm"`

## Multi-Account Spot Check

```bash
for u in jukan05 dnystedt trendforce nvidia; do
  echo "--- $u ---"
  curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=$u&recent=86400&limit=20&order_by=published_at_desc" \
    -H "X-API-Key: $OCTOARMS_API_KEY"
  echo
done
```

## Windowed Query Templates

Last 24 hours:

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=kyleichan&recent=86400&limit=50&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Absolute time window:

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=trendforce&published_after=2026-05-10T00:00:00Z&published_before=2026-05-12T00:00:00Z&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

## Troubleshooting Empty Results
If query returns empty data:
1. Confirm account has posted within `recent` window.
2. Increase window (for example `recent=259200` for 3 days).
3. Verify list members sync is applied for source `42`.
4. Trigger one manual run for `twitter_list_tweets:octoarms_hbm` source `43`, then re-query.
5. Confirm returned rows include `source_name: "octoarms_hbm"`.

## Notes
- This skill is scoped to list `2053711728324874698` only.
- Prefer account-based queries for operational checks.

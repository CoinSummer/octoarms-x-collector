---
name: octoarms-x-collector
description: Use when you need a standardized way to query monitored X list tweets (HBM/Tesla) from Octoarms contents API, verify recent collection results, and troubleshoot empty results.
---

# Query HBM / Tesla Tweets

## Overview
Use this skill to query list tweets through the Octoarms contents API in a consistent, repeatable way.

Supported lists in this skill:

| Topic | list_id | source_name | members source | tweets source | base tag |
|---|---|---|---:|---:|---|
| HBM | `2053711728324874698` | `octoarms_hbm` | `42` | `43` | `hbm` |
| Tesla | `2054109618323030125` | `octoarms_tesla` | `44` | `45` | `tesla` |

## When to Use
- You need to check whether HBM or Tesla list data is being collected.
- You need a quick JSON proof for one or more tracked accounts.
- You need to troubleshoot why a recent run appears to have no data.

## Prerequisites
- `OCTOARMS_BASE_URL` is set.
- `OCTOARMS_API_KEY` is set and valid.
- Target list member usernames are known (or fetched from source subscriptions).

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
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=<USERNAME>&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Expected signal in response items:
- `success: true`
- `source_name` should match your target list's `source_name`
- `username` matches query username

## How to Restrict to a Specific List

Current practical restriction strategy:
1. Query only usernames that are members of your target list.
2. Validate result rows contain the target `source_name`.

Reason:
- Contents API currently does not expose a stable, explicit `list_id` filter in query params.
- Therefore list scoping is done by account scope + response verification.

### Step A: Get usernames for target list

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner-inter/sources/<MEMBERS_SOURCE_ID>/subscriptions?provider=twitter&limit=500" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Read usernames from `data.items[].meta_json.username`.

### Step B: Query by username and assert `source_name`

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=<USERNAME>&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Check each returned row:
- `source_name == <TARGET_SOURCE_NAME>`

## Multi-Account Spot Check

```bash
for u <USER_1> <USER_2> <USER_3>; do
  echo "--- $u ---"
  curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=$u&recent=86400&limit=20&order_by=published_at_desc" \
    -H "X-API-Key: $OCTOARMS_API_KEY"
  echo
done
```

## Windowed Query Templates

Last 24 hours:

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=<USERNAME>&recent=86400&limit=50&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Absolute time window:

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&username=<USERNAME>&published_after=2026-05-10T00:00:00Z&published_before=2026-05-12T00:00:00Z&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

## Source Tag Filtering

Tier filter (any):

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&source_tags=tier_s&source_tags_mode=any&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Base tag + tier (all):

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&source_tags=<BASE_TAG>,tier_s&source_tags_mode=all&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Recommended `source_tags` pattern:
- `<BASE_TAG>` + `tier_s|tier_a|tier_b|tier_c|tier_d`

Common combinations:
- Tier + topic scope (example: `source_tags=<BASE_TAG>,tier_a&source_tags_mode=all`)
- Tier-only filter (example: `source_tags=tier_c&source_tags_mode=any`)

## Content Tag Filtering

Content tags (any):

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&tags=hbm,semiconductor&tags_mode=any&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Content tags (all):

```bash
curl -sS "$OCTOARMS_BASE_URL/api/v1/scanner/contents?type=tweet&source=twitter&tags=hbm,ai&tags_mode=all&recent=86400&limit=100&order_by=published_at_desc" \
  -H "X-API-Key: $OCTOARMS_API_KEY"
```

Note:
- `tags`/`tags_mode` filters content-level tags.
- `source_tags`/`source_tags_mode` filters source-subscription tags.

Current observed content tags (sample window):
- No content tags currently observed (`[]`).

Implication:
- `source_tags` is currently the reliable filter for tier slicing.
- `tags` filters will only become effective after content tagging pipeline writes content-level tags.

## Troubleshooting Empty Results
If query returns empty data:
1. Confirm account has posted within `recent` window.
2. Increase window (for example `recent=259200` for 3 days).
3. Verify list members sync is applied for the target members source.
4. Trigger one manual run for the target tweets source, then re-query.
5. Confirm returned rows include target `source_name`.

## Notes
- This skill supports both HBM and Tesla lists above.
- Prefer account-based queries for operational checks.

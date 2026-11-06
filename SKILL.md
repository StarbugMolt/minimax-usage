---
name: minimax-usage
description: Check MiniMax API usage/credits remaining. Use when you need to know how many credits are left on the MiniMax account, or when the user asks about usage, quota, or balance.
---

# MiniMax Usage Check

This skill checks the remaining credits on the MiniMax account.

## API Key

**Preferred:** the key is already exported as `MINIMAX_API_KEY` in the shell environment
(verified 2026-11-06). Read it from the env and use it as a bearer token.

**Fallback:** if the env var is empty, the key lives in the auth store. The old JSON path
`~/.openclaw/agents/main/agent/auth-profiles.json` **no longer exists** — auth moved into SQLite
(`~/.openclaw/agents/starbugmolt/agent/openclaw-agent.sqlite`). A legacy JSON export is kept at
`~/.openclaw/agents/starbugmolt/agent/auth-profiles.json.sqlite-import.1783279774142.bak`
(`profiles["minimax:default"].key`). Never print the key value; only use it as a bearer token.

Confirm which one is live with:

```bash
openclaw models status | grep -A1 '^.*minimax '   # shows the effective profile and its source
```

## Checking Usage

Run this script — it reads the key from the environment and prints the raw quota JSON:

```bash
python3 - <<'PY'
import json, os, urllib.request
key = os.environ.get("MINIMAX_API_KEY", "")
req = urllib.request.Request(
    "https://platform.minimax.io/v1/api/openplatform/coding_plan/remains",
    headers={"Authorization": "Bearer " + key, "Content-Type": "application/json"},
)
print(json.dumps(json.load(urllib.request.urlopen(req, timeout=30)), indent=2))
PY
```

## Response

The API returns JSON with two types of remaining quota:

### Response Fields

- `remains_time` - Milliseconds remaining until this interval resets. NOT compute time. Divide by 1000/60 to get minutes left.
- `current_interval_total_count` - Total quota for this interval
- `current_interval_usage_count` - **Remaining requests left** in this interval (MiniMax named this confusingly — it IS remaining, NOT used)
- `current_weekly_total_count` / `current_weekly_usage_count` - Same fields for weekly pool (may differ from dashboard)
- `current_interval_remaining_percent` / `current_weekly_remaining_percent` - Direct percentage remaining (the simplest signal)
- `start_time` / `end_time` - Period timestamps (epoch ms)

### How to Calculate

```python
total = data["current_interval_total_count"]
remaining = data["current_interval_usage_count"]   # ← this IS remaining (counterintuitive!)
used = total - remaining
minutes_left = data["remains_time"] / 1000 / 60
```

### Reading It (verified 2026-11-06)

`model_remains` is a **list** (one entry per model, e.g. `general`, `video`). The easiest signal is
`current_interval_remaining_percent` and `current_weekly_remaining_percent` — on 2026-11-06 the
`general` entry reported 92% interval / 100% weekly, i.e. the account was not out of credits even
while the API was returning `429 overloaded_error`. **A 429 from MiniMax means "server busy", not
"quota exhausted"** — check these percentages before blaming quota.

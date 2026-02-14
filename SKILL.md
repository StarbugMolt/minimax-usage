---
name: minimax-usage
description: Check MiniMax API usage/credits remaining. Use when you need to know how many credits are left on the MiniMax account, or when the user asks about usage, quota, or balance.
---

# MiniMax Usage Check

This skill checks the remaining credits on your MiniMax account.

## API Key

The API key is stored in `/home/starbugmolt/.openclaw/agents/main/agent/auth-profiles.json` under `profiles.minimax:default.key`.

## Checking Usage

Make a curl request to the MiniMax API:

```bash
curl --location 'https://platform.minimax.io/v1/api/openplatform/coding_plan/remains' \
  --header 'Authorization: Bearer <API_KEY>' \
  --header 'Content-Type: application/json'
```

Replace `<API_KEY>` with the actual API key from the auth-profiles.json file.

## Response

The API returns JSON with two types of remaining quota:

### Response Fields

- `remains_time` - Remaining compute time in **milliseconds**
- `current_interval_total_count` - Display denominator (not used)
- `current_interval_usage_count` - Requests remaining in current period
- `start_time` / `end_time` - Period timestamps (epoch ms)
- `model_name` - Model name (may show "MiniMax-M2" even when using M2.5)

### Example

```json
{
  "model_remains": [{
    "remains_time": 14731640,
    "current_interval_total_count": 1500,
    "current_interval_usage_count": 1429,
    "model_name": "MiniMax-M2"
  }]
}
```

Present the usage as:
- **Compute time remaining**: `remains_time / 1000 / 60` = minutes left
- **Requests remaining**: `current_interval_total_count - current_interval_usage_count`

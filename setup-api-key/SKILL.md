---
name: setup-api-key
description: Configure and validate a slng.ai Voice AI API key. Use when getting started with slng, when an API call returns 401, or when the user asks to "set up VOICEAI_API_KEY", "configure slng credentials", or "get a voiceai key".
license: MIT
---

# Setup slng API Key

Configure `VOICEAI_API_KEY` so the CLI, SDKs, and direct REST calls can authenticate against `https://api.slng.ai`.

## Handling rules

- If `$VOICEAI_API_KEY` is already set, use it silently. Do not ask the user to paste or confirm it.
- Never echo the key back in responses, logs, error messages, or shell history.
- Prefer `read -s VOICEAI_API_KEY && export VOICEAI_API_KEY` (terminal does not echo, key stays out of the conversation transcript) over asking the user to paste in chat. Only fall back to in-chat paste if the environment cannot be set before the session.
- After saving to `.env`, confirm `.env` is in `.gitignore` before continuing.

## Workflow

### 1. Check if a key is already configured

```bash
echo "$VOICEAI_API_KEY"
```

Also check a project `.env`:

```bash
grep -E '^VOICEAI_API_KEY=' .env 2>/dev/null
```

If a key is set, validate it (see step 3) and stop. Do not prompt the user.

### 2. If missing, obtain a key

Direct the user to the dashboard:

> Open https://slng.ai/dashboard/api-keys, create a new key, and copy it now. The key is shown only once.

Ask the user to paste the key.

### 3. Validate the key

Make one lightweight authenticated call. `GET /v1/agents` on the agents API is a free DB read — no TTS/STT credits consumed:

```bash
curl -sS -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  https://api.agents.slng.ai/v1/agents
```

- `200` — key is good.
- `401` — key is invalid. Ask the user to re-check and paste again. Allow one retry, then stop.

### 4. Persist the key

Ask the user where to save it:

**Project `.env`** (recommended for repos):

```bash
echo 'VOICEAI_API_KEY=...' >> .env
```

Make sure `.env` is in `.gitignore`.

**Shell rc** (for global use):

```bash
# zsh
echo 'export VOICEAI_API_KEY="..."' >> ~/.zshrc

# bash
echo 'export VOICEAI_API_KEY="..."' >> ~/.bashrc
```

Then `source ~/.zshrc` or open a new terminal.

**voiceai CLI config** (third option, used only by the CLI):

```bash
voiceai config set api_key "..."
```

Stored at `~/.config/voiceai/config.json`. Note: env vars override anything in this file.

To wipe local CLI state (config file, cached keys, legacy `~/.config/slng/` directory), run:

```bash
voiceai config reset --force
```

Useful when reconfiguring against a different workspace or before uninstalling — `brew uninstall` does not remove the config directory.

## Environment variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `VOICEAI_API_KEY` | Bearer token sent as `Authorization: Bearer <key>` | Yes |
| `VOICEAI_BASE_URL` | Override the API base URL (e.g. for staging) | No |

## Security

- Never commit keys to git. Always gitignore `.env`.
- Treat the key like a password. Rotate it from the dashboard if leaked.

## Common errors

- **401 Unauthorized** — key missing, malformed, or revoked. Re-check the dashboard.
- **403 Forbidden** — key is valid but lacks permission for that endpoint. Check the workspace it belongs to.
- **429 Too Many Requests** — rate limit. Back off and retry.

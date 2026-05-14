---
name: agents
description: Create, manage, and dispatch slng.ai voice agents. Use when the user wants to create a voice agent, list or update existing agents, dispatch an outbound phone call from an agent, start a web (browser) voice session, or work with the slng Agent Infra / Voice Agents API.
license: MIT
compatibility: Requires internet access and a slng.ai API key (VOICEAI_API_KEY).
---

# slng Voice Agents

Manage voice agents and place real phone calls through slng's Voice Agents API. The agents API lives on a different base than the unified TTS/STT API.

**Base URL:** `https://api.agents.slng.ai`
**Auth:** `Authorization: Bearer $VOICEAI_API_KEY`

> **Setup:** [`setup-api-key`](../setup-api-key/SKILL.md) configures `VOICEAI_API_KEY`. Install paths are in [`references/installation.md`](references/installation.md). To craft the agent's `system_prompt` and `greeting`, use the [`agent-prompt`](../agent-prompt/SKILL.md) skill.

## Quick Start: Create an agent and dispatch a call

### cURL

```bash
# 1. Create
AGENT_ID=$(curl -sS https://api.agents.slng.ai/v1/agents \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Hello Agent",
    "greeting": "Hi, thanks for calling Acme. How can I help you today?",
    "system_prompt": "[Identity]\n- You are a friendly receptionist for Acme.\n[Style]\n- Warm and concise.",
    "voice": "aura-2-luna-en",
    "model": "claude-haiku-4-5"
  }' | jq -r .id)

# 2. Dispatch an outbound call
curl https://api.agents.slng.ai/v1/agents/$AGENT_ID/calls \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"phone_number": "+14155551234"}'
```

### Python

```python
import os, requests

BASE = "https://api.agents.slng.ai/v1"
H = {"Authorization": f"Bearer {os.environ['VOICEAI_API_KEY']}"}

agent = requests.post(f"{BASE}/agents", headers=H, json={
    "name": "Hello Agent",
    "greeting": "Hi, thanks for calling Acme. How can I help you today?",
    "system_prompt": "[Identity]\n- You are a friendly receptionist for Acme.",
    "voice": "aura-2-luna-en",
    "model": "claude-haiku-4-5",
}).json()

requests.post(f"{BASE}/agents/{agent['id']}/calls", headers=H, json={
    "phone_number": "+14155551234",
})
```

### TypeScript

```typescript
const BASE = "https://api.agents.slng.ai/v1";
const h = {
  Authorization: `Bearer ${process.env.VOICEAI_API_KEY}`,
  "Content-Type": "application/json",
};

const agent = await fetch(`${BASE}/agents`, {
  method: "POST",
  headers: h,
  body: JSON.stringify({
    name: "Hello Agent",
    greeting: "Hi, thanks for calling Acme. How can I help you today?",
    system_prompt: "[Identity]\n- You are a friendly receptionist for Acme.",
    voice: "aura-2-luna-en",
    model: "claude-haiku-4-5",
  }),
}).then((r) => r.json());

await fetch(`${BASE}/agents/${agent.id}/calls`, {
  method: "POST",
  headers: h,
  body: JSON.stringify({ phone_number: "+14155551234" }),
});
```

## Endpoints at a glance

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/v1/agents` | Create agent |
| `GET` | `/v1/agents` | List agents |
| `GET` | `/v1/agents/{id}` | Get agent |
| `PATCH` | `/v1/agents/{id}` | Update agent (partial) |
| `PUT` | `/v1/agents/{id}` | Replace agent |
| `DELETE` | `/v1/agents/{id}` | Delete agent |
| `POST` | `/v1/agents/{id}/duplicate` | Duplicate agent |
| `POST` | `/v1/agents/{id}/calls` | Dispatch outbound call |
| `GET` | `/v1/agents/{id}/calls` | List calls for agent |
| `GET` | `/v1/agents/{id}/calls/{call_id}` | Get specific call |
| `POST` | `/v1/agents/{id}/web-sessions` | Create browser voice session |
| `POST` | `/v1/tools/executions` | Submit async tool execution result |

See [`references/managing-agents.md`](references/managing-agents.md) and [`references/calls-and-sessions.md`](references/calls-and-sessions.md) for full examples in all four paths.

## Agent config shape

Minimum:

```json
{
  "name": "Hello Agent",
  "greeting": "Hi, thanks for calling Acme. How can I help you today?",
  "system_prompt": "...",
  "voice": "aura-2-luna-en",
  "model": "claude-haiku-4-5"
}
```

Optional fields:

- `tools` — webhook tools the agent can call. See the [`agent-prompt`](../agent-prompt/SKILL.md) skill for the schema and LLM-vs-system webhook guidance.
- `template_defaults` — fallback values for `{{variable}}` placeholders in `greeting` / `system_prompt`.
- `stt_model` — override default STT model.
- `language` — primary language hint.

## Dispatching a call

Outbound calls require a phone number in E.164 format. Pass `arguments` to fill template variables:

```json
{
  "phone_number": "+14155551234",
  "arguments": {
    "customer_name": "Maria",
    "package_name": "Weekend Getaway"
  }
}
```

Constraints on `arguments`: max 32 keys, key ≤ 64 chars, value ≤ 1024 chars, combined payload ≤ 8192 chars.

## Web (browser) sessions

For embedding an agent in a website (no phone), create a web session and pass the returned LiveKit token to the browser:

```bash
curl https://api.agents.slng.ai/v1/agents/$AGENT_ID/web-sessions \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"arguments": {"customer_name": "Maria"}}'
```

Response includes a LiveKit `token` and `room_name`. The browser then connects with the standard LiveKit client.

## Error handling

| Status | Meaning |
|--------|---------|
| `400` | Bad request — usually invalid `system_prompt`, `voice`, or `model` |
| `401` | Invalid API key |
| `404` | Agent or call not found |
| `409` | Conflict (e.g. duplicate name) |
| `422` | Validation error — error body lists invalid fields |
| `429` | Rate limit |

## See also

- [`references/installation.md`](references/installation.md)
- [`references/managing-agents.md`](references/managing-agents.md)
- [`references/calls-and-sessions.md`](references/calls-and-sessions.md)
- [`agent-prompt`](../agent-prompt/SKILL.md) — generate the `greeting`, `system_prompt`, template variables, and tools to feed into agent creation

# slng Skills

A pack of agent skills for the [slng.ai](https://slng.ai) Voice AI API. Drop any folder into your agent's skills directory (e.g. `~/.claude/skills/`) and the agent gains that capability — text-to-speech, transcription, voice-agent management, or prompt authoring.

## Skills

| Skill | Purpose |
|-------|---------|
| [`setup-api-key`](./setup-api-key) | Configure and validate `VOICEAI_API_KEY` |
| [`text-to-speech`](./text-to-speech) | Generate speech from text (Deepgram Aura, Rime Arcana, Cartesia, etc.) |
| [`speech-to-text`](./speech-to-text) | Transcribe audio (Deepgram Nova, Sarvam Saaras, Soniox, Reson8) |
| [`agent-prompt`](./agent-prompt) | Build a complete voice agent prompt ready to paste into the SLNG Agent Builder |
| [`agents`](./agents) | Create, manage, and dispatch slng voice agents |

## Integration paths

Every skill shows the same task across four paths:

- **CLI** — `voiceai` (install with `brew install slng-ai/tap/voiceai`, `curl -fsSL https://docs.slng.ai/install.sh | sh`, or `npm i -g voiceai-cli`)
- **Python SDK** — `pip install voiceai-sdk`
- **TypeScript SDK** — `npm install voiceai-sdk`
- **REST / WebSocket** — direct `curl` and `wscat` against `https://api.slng.ai`

## Setup

```bash
cp .env.example .env
# edit .env and add your VOICEAI_API_KEY
```

Get a key from the [slng dashboard](https://slng.ai/dashboard/api-keys). See the [`setup-api-key`](./setup-api-key) skill for the full flow.

## License

MIT

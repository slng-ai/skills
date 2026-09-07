<p align="center">
  <a href="https://slng.ai">
    <img src="https://www.datocms-assets.com/182222/1763142213-logo-lg.svg" alt="slng.ai" height="64" />
  </a>
</p>

<h1 align="center">SLNG skills</h1>

<p align="center">
  <a href="https://skills.sh/slng-ai/skills">
    <img src="https://skills.sh/b/slng-ai/skills" alt="skills count" />
  </a>
</p>

A pack of agent skills for the [slng.ai](https://slng.ai) Voice AI API — text-to-speech, transcription, voice-agent management, and prompt authoring.

Drop any folder into your agent's skills directory (e.g. `~/.claude/skills/`) and the agent gains that capability.

## Install

Install all skills into your agent's skills directory:

```bash
npx skills add slng-ai/skills --all          # install everything
npx skills add slng-ai/skills text-to-speech # install one skill
```

**Manual install:** copy any skill folder (e.g. `text-to-speech/`) into `~/.claude/skills/`.

> Requires Node ≥ 18 for `npx skills`.

## Setup

```bash
cp .env.example .env
# edit .env and add your VOICEAI_API_KEY
```

Get a key from the [SLNG dashboard](https://app.slng.ai/api-keys). See the [`setup-api-key`](./setup-api-key) skill for the full flow.

## Skills

| Skill | Purpose |
|-------|---------|
| [`setup-api-key`](./setup-api-key) | Configure and validate `VOICEAI_API_KEY` |
| [`text-to-speech`](./text-to-speech) | Generate speech from text (Deepgram Aura, Rime Arcana, Cartesia, etc.) |
| [`speech-to-text`](./speech-to-text) | Transcribe audio (Deepgram Nova, Sarvam Saaras, Soniox, Reson8) |
| [`agent-prompt`](./agent-prompt) | Build a complete voice agent prompt ready to paste into the SLNG Agent Builder |
| [`agents`](./agents) | Create, manage, and dispatch SLNG voice agents |
| [`livekit-migration`](./livekit-migration) | Migrate an existing LiveKit Agents Python project to SLNG hosted STT/TTS |
| [`pipecat-migration`](./pipecat-migration) | Migrate an existing Pipecat Python project to SLNG hosted STT/TTS |
| [`custom-migration`](./custom-migration) | Migrate a custom Python/JS voice project to SLNG hosted STT/TTS and optional Context Router |

## Integration paths

Each skill shows the same workflow in four flavors — pick whichever fits your stack:

- **CLI** — [`voiceai`](https://www.npmjs.com/package/voiceai-cli)
  - `npm i -g voiceai-cli`
  - `brew install slng-ai/tap/voiceai`
  - `curl -fsSL https://docs.slng.ai/install.sh | sh`
- **Python SDK** — [`pip install voiceai-sdk`](https://pypi.org/project/voiceai-sdk/)
- **TypeScript SDK** — [`npm install voiceai-sdk`](https://www.npmjs.com/package/voiceai-sdk)
- **REST / WebSocket** — direct `curl` and `wscat` against `https://api.slng.ai`

---

Licensed under [MIT](./LICENSE).

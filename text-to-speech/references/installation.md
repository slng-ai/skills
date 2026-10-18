# Installation

All paths authenticate with `VOICEAI_API_KEY` and send the project id from `SLNG_PROJECT_ID` in the `X-SLNG-Project-Id` header. See [`setup-api-key`](../../setup-api-key/SKILL.md) to configure it.

## CLI

```bash
# Homebrew (macOS / Linux)
brew install slng-ai/tap/voiceai

# Install script
curl -fsSL https://docs.slng.ai/install.sh | sh

# npm (cross-platform)
npm install -g voiceai-cli
```

Then:

```bash
export VOICEAI_API_KEY="..."        # or use `voiceai config set apiKey ...`
export SLNG_PROJECT_ID="..."        # or pass `--project <id>`
voiceai tts "hello"                  # plays + saves to $TMPDIR/voiceai-tts/
voiceai tts "save me" --out hi.mp3   # save to a specific path
```

Run `voiceai` with no arguments to open the interactive TUI.

## Python

```bash
pip install voiceai-sdk
```

```python
import os
from voiceai import VoiceAI

client = VoiceAI(
    api_key=os.environ["VOICEAI_API_KEY"],
    default_headers={"X-SLNG-Project-Id": os.environ["SLNG_PROJECT_ID"]},
)

audio = client.tts.create(
    model="slng/deepgram/aura:2-en",
    text="Hello from Python!",
    voice="aura-2-luna-en",
)

with open("hello.wav", "wb") as f:
    f.write(audio.read())
```

If you'd rather avoid the SDK, plain `requests` works:

```python
import os, requests
r = requests.post(
    "https://api.slng.ai/v1/tts/slng/deepgram/aura:2",
    headers={
        "Authorization": f"Bearer {os.environ['VOICEAI_API_KEY']}",
        "X-SLNG-Project-Id": os.environ["SLNG_PROJECT_ID"],
    },
    json={"text": "Hello", "model": "aura-2-luna-en"},
)
open("hello.wav", "wb").write(r.content)
```

## TypeScript

```bash
npm install voiceai-sdk
```

```typescript
import { VoiceAI } from "voiceai-sdk";

const client = new VoiceAI({
  apiKey: process.env.VOICEAI_API_KEY!,
  defaultHeaders: { "X-SLNG-Project-Id": process.env.SLNG_PROJECT_ID! },
});

const audio = await client.tts.create({
  model: "slng/deepgram/aura:2-en",
  text: "Hello from TypeScript!",
  voice: "aura-2-luna-en",
});

await Bun.write("hello.wav", await audio.arrayBuffer());
```

With plain `fetch`:

```typescript
const r = await fetch("https://api.slng.ai/v1/tts/slng/deepgram/aura:2", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.VOICEAI_API_KEY}`,
    "X-SLNG-Project-Id": process.env.SLNG_PROJECT_ID!,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ text: "Hello", model: "aura-2-luna-en" }),
});
const buf = Buffer.from(await r.arrayBuffer());
require("fs").writeFileSync("hello.wav", buf);
```

## cURL

No install needed.

```bash
export VOICEAI_API_KEY="..."
export SLNG_PROJECT_ID="..."

curl https://api.slng.ai/v1/tts/slng/deepgram/aura:2 \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "X-SLNG-Project-Id: $SLNG_PROJECT_ID" \
  -H "Content-Type: application/json" \
  -d '{"text":"Hello","model":"aura-2-luna-en"}' \
  --output hello.wav
```

## Environment variable reference

| Variable | Purpose |
|----------|---------|
| `VOICEAI_API_KEY` | Bearer token. Required. |
| `SLNG_PROJECT_ID` | Project UUID, sent as the `X-SLNG-Project-Id` header. Picks the project the request is billed and limited under. |
| `VOICEAI_BASE_URL` | Override the base URL (default `https://api.slng.ai`). Useful for staging. |

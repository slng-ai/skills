# Installation

All paths authenticate with `VOICEAI_API_KEY` and send the project id from `SLNG_PROJECT_ID` in the `X-SLNG-Project-Id` header. See [`setup-api-key`](../../setup-api-key/SKILL.md).

## CLI

```bash
# Homebrew (macOS / Linux)
brew install slng-ai/tap/voiceai

# Install script
curl -fsSL https://docs.slng.ai/install.sh | sh

# npm (cross-platform)
npm install -g voiceai-cli
```

> Mic capture (`voiceai stt --stream`) uses `sox` for audio I/O. The Homebrew formula recommends it automatically; install manually with `brew install sox` if you skipped it.

```bash
export SLNG_PROJECT_ID="..."                        # or pass `--project <id>`
voiceai stt audio.wav -m slng/deepgram/nova:3-en
voiceai stt --stream                                 # mic
```

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

with open("audio.wav", "rb") as f:
    result = client.stt.transcribe(
        model="slng/deepgram/nova:3-en",
        audio=f,
    )

print(result.transcript)
```

With plain `requests`:

```python
import os, requests
with open("audio.wav", "rb") as f:
    r = requests.post(
        "https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en",
        headers={
            "Authorization": f"Bearer {os.environ['VOICEAI_API_KEY']}",
            "X-SLNG-Project-Id": os.environ["SLNG_PROJECT_ID"],
        },
        files={"audio": f},
    )
print(r.json())
```

## TypeScript

```bash
npm install voiceai-sdk
```

```typescript
import { VoiceAI } from "voiceai-sdk";
import { createReadStream } from "fs";

const client = new VoiceAI({
  apiKey: process.env.VOICEAI_API_KEY!,
  defaultHeaders: { "X-SLNG-Project-Id": process.env.SLNG_PROJECT_ID! },
});

const result = await client.stt.transcribe({
  model: "slng/deepgram/nova:3-en",
  audio: createReadStream("audio.wav"),
});

console.log(result.transcript);
```

With plain `fetch`:

```typescript
import { readFileSync } from "fs";

const form = new FormData();
form.append("audio", new Blob([readFileSync("audio.wav")]), "audio.wav");

const r = await fetch("https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.VOICEAI_API_KEY}`,
    "X-SLNG-Project-Id": process.env.SLNG_PROJECT_ID!,
  },
  body: form,
});
console.log(await r.json());
```

## cURL

```bash
curl https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "X-SLNG-Project-Id: $SLNG_PROJECT_ID" \
  -F "audio=@audio.wav"
```

## Environment variable reference

| Variable | Purpose |
|----------|---------|
| `VOICEAI_API_KEY` | Bearer token. Required. |
| `SLNG_PROJECT_ID` | Project UUID, sent as the `X-SLNG-Project-Id` header. Picks the project the request is billed and limited under. |
| `VOICEAI_BASE_URL` | Override base URL (default `https://api.slng.ai`). |

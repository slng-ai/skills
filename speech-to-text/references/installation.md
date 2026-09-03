# Installation

All paths authenticate with `VOICEAI_API_KEY`. See [`setup-api-key`](../../setup-api-key/SKILL.md).

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
voiceai stt audio.wav -m slng/deepgram/nova:3-en
voiceai stt --stream                                 # mic
```

## Python

```bash
pip install voiceai-sdk
```

```python
import os
from pathlib import Path
from voiceai_sdk import Slng

client = Slng(api_key=os.environ["VOICEAI_API_KEY"])

transcript = client.speech_to_text.create(
    model_variant="slng/deepgram/nova:3-en",
    audio=Path("audio.wav"),
)

print(transcript.text)
```

With plain `requests`:

```python
import os, requests
with open("audio.wav", "rb") as f:
    r = requests.post(
        "https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en",
        headers={"Authorization": f"Bearer {os.environ['VOICEAI_API_KEY']}"},
        files={"audio": f},
    )
print(r.json())
```

## TypeScript

```bash
npm install voiceai-sdk
```

The `voiceai-sdk` npm package exports a `Slng` client, but its methods are generated per model
(e.g. `client.nova3`), so check the installed package's types for the exact call shape. The HTTP
endpoint below is stable and simplest:

```typescript
import { readFileSync } from "fs";

const form = new FormData();
form.append("audio", new Blob([readFileSync("audio.wav")]), "audio.wav");

const r = await fetch("https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en", {
  method: "POST",
  headers: { Authorization: `Bearer ${process.env.VOICEAI_API_KEY}` },
  body: form,
});
console.log(await r.json());
```

## cURL

```bash
curl https://api.slng.ai/v1/stt/slng/deepgram/nova:3-en \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -F "audio=@audio.wav"
```

## Environment variable reference

| Variable | Purpose |
|----------|---------|
| `VOICEAI_API_KEY` | Bearer token. Required. |
| `VOICEAI_BASE_URL` | Override base URL (default `https://api.slng.ai`). |

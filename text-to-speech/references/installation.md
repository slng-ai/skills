# Installation

All paths authenticate with `VOICEAI_API_KEY`. See [`setup-api-key`](../../setup-api-key/SKILL.md) to configure it.

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
from voiceai_sdk import Slng

client = Slng(api_key=os.environ["VOICEAI_API_KEY"])

audio = client.text_to_speech.create(
    model_variant="slng/deepgram/aura:2-en",
    text="Hello from Python!",
    voice="aura-2-luna-en",
)

audio.write_to_file("hello.wav")
```

If you'd rather avoid the SDK, plain `requests` works:

```python
import os, requests
r = requests.post(
    "https://api.slng.ai/v1/tts/slng/deepgram/aura:2",
    headers={"Authorization": f"Bearer {os.environ['VOICEAI_API_KEY']}"},
    json={"text": "Hello", "model": "aura-2-luna-en"},
)
open("hello.wav", "wb").write(r.content)
```

## TypeScript

```bash
npm install voiceai-sdk
```

The `voiceai-sdk` npm package exports a `Slng` client, but its methods are generated per model
(e.g. `client.aura2Deepgram`), so check the installed package's types for the exact call shape. The
HTTP endpoint below is stable and simplest for one-shot synthesis:

```typescript
const r = await fetch("https://api.slng.ai/v1/tts/slng/deepgram/aura:2", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.VOICEAI_API_KEY}`,
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

curl https://api.slng.ai/v1/tts/slng/deepgram/aura:2 \
  -H "Authorization: Bearer $VOICEAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text":"Hello","model":"aura-2-luna-en"}' \
  --output hello.wav
```

## Environment variable reference

| Variable | Purpose |
|----------|---------|
| `VOICEAI_API_KEY` | Bearer token. Required. |
| `VOICEAI_BASE_URL` | Override the base URL (default `https://api.slng.ai`). Useful for staging. |

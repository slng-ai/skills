# SLNG SDK and Context Router Reference

Use this reference for concrete wiring once project discovery has identified the language, runtime,
and selected stages.

## Package Names

- Python STT/TTS SDK: `voiceai-sdk`
- JavaScript/TypeScript STT/TTS SDK: `voiceai-sdk`
- LLM: use an OpenAI-compatible client with an SLNG Router `base_url`/`baseURL`, provisioned org
  config, `slng/auto`, and required agent/session headers

Do not use fake packages such as `slng-sdk`, or fake classes such as `VoiceAgent`, `STT`, `TTS`,
`LLM`, or `Intelligence`, unless the target project already defines those names.

## Python STT/TTS

Prefer the project's existing sync/async style.

```python
import os
from pathlib import Path
from voiceai_sdk import Slng

client = Slng(api_key=os.environ["SLNG_API_KEY"])

transcript = client.speech_to_text.create(
    model_variant="slng/deepgram/nova:3-en",
    audio=Path("audio.wav"),
)

audio = client.text_to_speech.create(
    model_variant="slng/deepgram/aura:2-en",
    text=text,
    voice="aura-2-luna-en",
)
```

The result objects expose `transcript.text` and `audio.write_to_file(path)`. The import module is
`voiceai_sdk` (underscore), not `voiceai`. For async projects:

```python
import os
from voiceai_sdk import AsyncSlng

client = AsyncSlng(api_key=os.environ["SLNG_API_KEY"])
```

Use streaming SDK helpers only when replacing an existing streaming seam. Keep the current event names
or adapter return shape so callers do not need to change.

## JavaScript/TypeScript STT/TTS

For server-side Node/Bun projects, the `voiceai-sdk` npm package exports a `Slng` client:

```ts
import { Slng } from "voiceai-sdk";

const client = new Slng({ apiKey: process.env.SLNG_API_KEY });
```

The TS SDK's request methods are generated per model (e.g. `client.nova3`, `client.aura2Deepgram`)
rather than a single `client.tts` / `client.stt` namespace, and its surface diverges from the Python
client — check the installed package's types (`voiceai-sdk/models`) for the exact call shape, or use
the HTTP endpoints directly against `https://api.slng.ai/v1/{tts,stt}/{provider}/{model}:{variant}`.
Do not expose `SLNG_API_KEY` in browser bundles; route browser audio/text through a server-side endpoint.

## Optional Context Router

Only migrate the LLM when selected and SLNG has provisioned org/router configuration for the customer.
Prefer the project's existing OpenAI-compatible client if present. Otherwise add the standard OpenAI
SDK for the project language.

Regional router base URLs:

| Region | Base URL |
|--------|----------|
| Europe | `https://eu.context-router.slng.ai/v1` |
| United States | `https://us.context-router.slng.ai/v1` |
| India | `https://india.context-router.slng.ai/v1` |
| Indonesia | `https://indonesia.context-router.slng.ai/v1` |

Python example:

```python
import os
from openai import OpenAI

llm = OpenAI(
    api_key=os.environ["SLNG_API_KEY"],
    base_url="https://us.context-router.slng.ai/v1",
    default_headers={
        "X-Slng-Agent-Id": agent_id,
        "X-Slng-Session-Id": session_id,
    },
)

response = llm.chat.completions.create(
    model="slng/auto",
    messages=messages,
)
```

JavaScript/TypeScript example:

```ts
import OpenAI from "openai";

const llm = new OpenAI({
  apiKey: process.env.SLNG_API_KEY,
  baseURL: "https://us.context-router.slng.ai/v1",
  defaultHeaders: {
    "X-Slng-Agent-Id": agentId,
    "X-Slng-Session-Id": sessionId,
  },
});

const response = await llm.chat.completions.create({
  model: "slng/auto",
  messages,
});
```

Use `slng/auto` as the only code-level model. Do not place provider/catalog model ids in customer
code; SLNG configures provider models, customer provider API keys, routing policy, cache behavior,
and other router parameters in org configuration. If org configuration is not ready, stop before LLM
edits and report that SLNG must provision it before verification.

Use stable project identifiers for `agent_id`/`agentId` and `session_id`/`sessionId`, such as an app
agent id plus conversation, room, call, or session id. Do not generate random IDs per request. Preserve
streaming behavior if the old LLM call streamed.

## Region and Model Notes

For STT/TTS, use the selected model ids from the user's prompt or UI. Keep the existing language and
voice when they are valid for the selected SLNG model. For LLM, region is controlled by the router
base URL. For STT/TTS region pinning, only add region/base URL options that the current SDK docs or
project configuration confirm.

# slng LiveKit pipeline reference

Models, voices, regions, and the exact wire-up for migrating a LiveKit agent's STT and TTS to the
slng plugin. Verified against `livekit-plugins-slng` (import `from livekit.plugins import slng`).

> The plugin exposes `slng.STT` and `slng.TTS` only. There is **no `LLM` and no `Intelligence`
> class** — the agent's LLM stays on its current provider.

## Install and import

```bash
uv add livekit-plugins-slng
```

```python
from livekit.plugins import slng
```

The plugin reads `SLNG_API_KEY` from the environment automatically. Pass `api_key=os.environ["SLNG_API_KEY"]`
explicitly if you want it visible at the call site — never inline the key.

## STT — `slng.STT`

```python
slng.STT(
    model="deepgram/nova:3",     # default; provider/model:variant
    language="en",               # default
    region_override=None,        # e.g. "eu-north-1", or a priority list
    # api_key, slng_base_url="api.slng.ai", sample_rate, encoding,
    # enable_partial_transcripts, enable_diarization, vad_threshold, ... also available
)
```

## TTS — `slng.TTS`

```python
slng.TTS(
    model="deepgram/aura:2",     # default; provider/model:variant
    voice="default",             # voice id for the chosen model
    language="en",
    region_override=None,
    speed=1.0,
    # api_key, slng_base_url="api.slng.ai", sample_rate, ... also available
)
```

## Supported models

Models use a `provider/model:variant` id. The `slng/`-prefixed ids are SLNG-hosted; the others route a
provider through SLNG. This list reflects the SLNG catalog — confirm the current set and exact ids in the
[SLNG docs](https://docs.slng.ai/agents/livekit-plugin) / dashboard, as it changes over time.

**STT (`slng.STT`)**

| Model id | Provider | Notes |
|----------|----------|-------|
| `slng/deepgram/nova:3-en` | Deepgram | SLNG-hosted Nova 3, lowest latency. `-multi` variant for auto language |
| `deepgram/nova:2` | Deepgram | Nova 2, lower cost, 36 languages |
| `soniox/speech-ai:rt-v4` | Soniox | Real-time, diarisation, 60+ languages |
| `reson8/reson8stt:v1` | Reson8 | 9 European locales, telephony-tuned |
| `sarvam/saaras:v3` | Sarvam AI | Indian + European languages (24 locales) |

**TTS (`slng.TTS`)** — each model has its own voice ids.

| Model id | Provider | Notes |
|----------|----------|-------|
| `slng/rime/arcana:3-en` | Rime | SLNG-hosted Arcana v3, emotional prosody, low TTFB |
| `slng/deepgram/aura:2-en` | Deepgram | SLNG-hosted Aura 2, pairs natively with Nova 3 STT |
| `cartesia/sonic:3` | Cartesia | Sonic 3, WebSocket streaming, voice cloning, 40+ languages |
| `murf/murftts:falcon` | Murf | Studio quality, 16 locales |
| `kugelaudio/kugel:1-turbo` | KugelAudio | European languages, 26 locales |
| `soniox/tts-rt:v1` | Soniox | Real-time, 50+ languages |
| `sarvam/bulbul:v3` | Sarvam AI | 11 Indian-language locales |

(There is no LLM list — the plugin has no LLM; the LLM stays on the orchestrator.)

Like-for-like mapping from the standard starter — keep the same provider, routed through slng:

| Before (LiveKit Inference) | After (slng) |
|----------------------------|--------------|
| `deepgram/nova-3` | `deepgram/nova:3` |
| `cartesia/sonic-3` (voice `<id>`) | `cartesia/sonic:3` (same voice id) |

## Regions

Pin a region with `region_override` (string or priority list). If unset, the closest region is
auto-selected. Available regions include `us-east-1`, `eu-north-1`, `ap-south-1`, `ap-southeast-2`,
`asia-south1`, `asia-southeast2`, `australia-southeast1`. Not every model is deployed in every region.

```python
slng.STT(model="deepgram/nova:3", region_override="eu-north-1")
```

## Where each stage lives in the starter

In the standard `agent-starter-python` layout:

- **STT and TTS** are on the `AgentSession(stt=..., tts=...)` — these are what slng replaces.
- **LLM** is on the `Agent` subclass (`Agent.__init__(llm=...)`) and **stays** — the plugin has no LLM.
- **VAD** (`silero`), **turn detection** (`MultilingualModel`), and any **noise cancellation** plugin
  (e.g. `ai_coustics`) stay as they are unless the user opts to remove them.

## Before / after — standard starter

**Before** (LiveKit Inference):

```python
from livekit.agents import inference
# ... in AgentSession(...):
stt=inference.STT(model="deepgram/nova-3", language="multi"),
tts=inference.TTS(model="cartesia/sonic-3", voice="9626c31c-bec5-4cca-baa8-f8ba9e84c8bc"),
```

**After** (slng). Add the import once at the top of the file:

```python
from livekit.plugins import slng
```

In `AgentSession(...)`, replace STT and TTS (leave `vad`, `turn_detection`, and
`preemptive_generation` as they are, and leave the LLM on the `Agent`):

```python
stt=slng.STT(model="deepgram/nova:3", language="multi"),
tts=slng.TTS(model="cartesia/sonic:3", voice="9626c31c-bec5-4cca-baa8-f8ba9e84c8bc"),
```

Add `region_override="eu-north-1"` (or your region) to either call for data residency.

Keep `agent.py` as the entrypoint — the Dockerfile runs `uv run src/agent.py start`.

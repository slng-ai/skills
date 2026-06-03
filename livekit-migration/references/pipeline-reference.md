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

## Models

Models use a `provider/model:variant` id. Common choices (confirm the live catalog at
<https://docs.slng.ai/agents/livekit-plugin> before using a model you have not run):

| Stage | Example model ids | Notes |
|-------|-------------------|-------|
| STT | `deepgram/nova:3`, `slng/deepgram/nova:3-en`, `slng/deepgram/nova:3-multi`, `soniox/speech-ai-rt:v4` | `nova:3` is the default; `-multi` for auto language |
| TTS | `deepgram/aura:2`, `cartesia/sonic:3`, `rime/arcana:v3`, `sarvam/bulbul:v3`, `kugelaudio/kugel:2` | Each model has its own voice ids; Rime requires a `voice` per language |

Like-for-like mapping from the standard starter:

| Before (LiveKit Inference) | After (slng) |
|----------------------------|--------------|
| `deepgram/nova-3` | `deepgram/nova:3` |
| `cartesia/sonic-3` (voice `<id>`) | `slng/deepgram/aura:2-en`, voice `aura-2-thalia-en` (see known issue) |

> **Known issue:** routing Cartesia TTS through the plugin (`cartesia/sonic:3`) currently fails with
> `400 Bad Request - Invalid Cartesia-Version header`. Use a Deepgram Aura voice
> (`slng/deepgram/aura:2-en`) until that is fixed upstream. STT (`deepgram/nova:3`) works fine.

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
tts=slng.TTS(model="slng/deepgram/aura:2-en", voice="aura-2-thalia-en"),
```

Add `region_override="eu-north-1"` (or your region) to either call for data residency.

Keep `agent.py` as the entrypoint — the Dockerfile runs `uv run src/agent.py start`.

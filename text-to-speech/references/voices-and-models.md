# Voices and Models

slng routes to many TTS providers through one endpoint shape. Pick a **model** (URL path) and then a **voice** (request body `model` field, confusingly).

## Listing what's deployed

The CLI lists the live catalog (★ = SLNG-hosted):

```bash
voiceai models --tts
voiceai voices --model slng/deepgram/aura:2-en
```

> The `GET /v1/catalog/models` endpoint lists the full model catalog, but it has no
> active/deployed filter — it can't tell you which models are actually callable in your region. So
> the CLI above (`voiceai models` / `voiceai voices`) remains the authority for what's deployed.
> There is no `/v1/voices` endpoint. If neither is possible, just attempt a short synthesis with the
> candidate model/voice — the request itself is the availability check.

## Deployed TTS models (illustrative — not the live list)

`slng/`-prefixed ids are SLNG-hosted; the others route a provider through SLNG. This table drifts —
run `voiceai models --tts` for the live deployed set.

| Model id | Provider | Notes |
|----------|----------|-------|
| `slng/deepgram/aura:2-en` | Deepgram | SLNG-hosted Aura 2, common default |
| `slng/rime/arcana:3-en` | Rime | SLNG-hosted Arcana v3, high emotional range |
| `slng/rime/coda:0-id` | Rime | SLNG-hosted Coda v0, Indonesian |
| `cartesia/sonic:3` | Cartesia | Sonic 3, WebSocket streaming, ultra-low latency |
| `deepgram/aura:2` | Deepgram | Provider-routed Aura 2 |
| `elevenlabs/eleven-flash:2` | ElevenLabs | Flash 2 |
| `elevenlabs/eleven-flash:2.5` | ElevenLabs | Flash 2.5 |
| `elevenlabs/eleven-multilingual:2` | ElevenLabs | Multilingual 2 |
| `elevenlabs/eleven:3` | ElevenLabs | Eleven 3 |
| `kugelaudio/kugel:1` | KugelAudio | Kugel v1 |
| `kugelaudio/kugel:2` | KugelAudio | Kugel 2, studio quality |
| `kugelaudio/kugel:2-turbo` | KugelAudio | Kugel 2 Turbo |
| `murf/murftts:falcon` | Murf | Falcon, realistic brand-friendly voices |
| `sarvam/bulbul:v3` | Sarvam AI | Hindi, Tamil, Telugu, Marathi, Kannada |
| `soniox/tts-rt:v1` | Soniox | Real-time streaming |

## Recommended voices

### Deepgram Aura 2 — English feminine

`aura-2-luna-en` (warm, friendly — common default), `aura-2-thalia-en` (clear), `aura-2-asteria-en`, `aura-2-athena-en` (articulate), `aura-2-aurora-en`, `aura-2-callista-en`, `aura-2-cordelia-en`, `aura-2-cora-en`, `aura-2-delia-en`, `aura-2-electra-en`, `aura-2-harmonia-en`, `aura-2-helena-en`, `aura-2-hera-en`, `aura-2-iris-en`, `aura-2-juno-en`, `aura-2-minerva-en`, `aura-2-ophelia-en`, `aura-2-pandora-en`, `aura-2-phoebe-en`, `aura-2-selene-en`, `aura-2-theia-en`, `aura-2-vesta-en`.

### Deepgram Aura 2 — English masculine

`aura-2-orion-en` (default), `aura-2-apollo-en`, `aura-2-arcas-en`, `aura-2-aries-en`, `aura-2-atlas-en`, `aura-2-draco-en`, `aura-2-hermes-en`, `aura-2-hyperion-en`, `aura-2-janus-en`, `aura-2-jupiter-en`, `aura-2-mars-en`, `aura-2-neptune-en`, `aura-2-odysseus-en`, `aura-2-orpheus-en`, `aura-2-pluto-en`, `aura-2-saturn-en`, `aura-2-zeus-en`.

### Other providers

- **Rime Arcana v3** — `slng/rime/arcana:3-en`. See `voiceai voices --model slng/rime/arcana:3-en` for the catalog.
- **Cartesia Sonic 3** — `cartesia/sonic:3`. WebSocket-only; Cartesia voice ids (UUIDs) carry over.
- **Sarvam Bulbul v3** — `sarvam/bulbul:v3`. Hindi, Tamil, Telugu, Kannada, Marathi.
- **Kugel** — `kugelaudio/kugel:1`, `kugelaudio/kugel:2`, `kugelaudio/kugel:2-turbo`. WebSocket-only.
- **Murf Falcon** — `murf/murftts:falcon`. WebSocket-only.
- **Soniox TTS** — `soniox/tts-rt:v1`.

## Models by region

Some models are deployed only in certain regions. The CLI auto-routes; explicit override:

```bash
voiceai tts "..." --region eu-north-1
voiceai tts "..." --world-part eu
```

Available regions: `ap-south-1`, `ap-southeast-2`, `asia-south1`, `asia-southeast2`, `australia-southeast1`, `eu-north-1`, `us-east-1`.

Full per-region availability: https://docs.slng.ai (see Models by Region).

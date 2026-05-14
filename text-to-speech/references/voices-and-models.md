# Voices and Models

slng routes to many TTS providers through one endpoint shape. Pick a **model** (URL path) and then a **voice** (request body `model` field, confusingly).

## Listing what's deployed

### CLI

```bash
voiceai models --tts
voiceai voices --model slng/deepgram/aura:2-en
```

### Python

```python
from voiceai import VoiceAI

client = VoiceAI()
models = client.models.list(modality="tts")
for m in models:
    print(m.id, m.languages)

voices = client.voices.list(model="slng/deepgram/aura:2-en")
for v in voices:
    print(v.id, v.gender, v.description)
```

### TypeScript

```typescript
import { VoiceAI } from "voiceai-sdk";

const client = new VoiceAI();
const models = await client.models.list({ modality: "tts" });
const voices = await client.voices.list({ model: "slng/deepgram/aura:2-en" });
```

### cURL

```bash
curl https://api.slng.ai/v1/models?modality=tts \
  -H "Authorization: Bearer $VOICEAI_API_KEY"

curl "https://api.slng.ai/v1/voices?model=slng/deepgram/aura:2-en" \
  -H "Authorization: Bearer $VOICEAI_API_KEY"
```

## Recommended voices

### Deepgram Aura 2 — English feminine

`aura-2-luna-en` (warm, friendly — common default), `aura-2-thalia-en` (clear), `aura-2-asteria-en`, `aura-2-athena-en` (articulate), `aura-2-aurora-en`, `aura-2-callista-en`, `aura-2-cordelia-en`, `aura-2-cora-en`, `aura-2-delia-en`, `aura-2-electra-en`, `aura-2-harmonia-en`, `aura-2-helena-en`, `aura-2-hera-en`, `aura-2-iris-en`, `aura-2-juno-en`, `aura-2-minerva-en`, `aura-2-ophelia-en`, `aura-2-pandora-en`, `aura-2-phoebe-en`, `aura-2-selene-en`, `aura-2-theia-en`, `aura-2-vesta-en`.

### Deepgram Aura 2 — English masculine

`aura-2-orion-en` (default), `aura-2-apollo-en`, `aura-2-arcas-en`, `aura-2-aries-en`, `aura-2-atlas-en`, `aura-2-draco-en`, `aura-2-hermes-en`, `aura-2-hyperion-en`, `aura-2-janus-en`, `aura-2-jupiter-en`, `aura-2-mars-en`, `aura-2-neptune-en`, `aura-2-odysseus-en`, `aura-2-orpheus-en`, `aura-2-pluto-en`, `aura-2-saturn-en`, `aura-2-zeus-en`.

### Deepgram Aura 2 — Spanish

Feminine: `aura-2-carina-es`, `aura-2-celeste-es`, `aura-2-diana-es`, `aura-2-estrella-es`, `aura-2-selena-es`.
Masculine: `aura-2-sirio-es`, `aura-2-nestor-es`, `aura-2-alvaro-es`, `aura-2-aquila-es`, `aura-2-javier-es`.

### Other providers

- **Rime Arcana v3** — `rime/arcana:3-en`, `rime/arcana:3-hi`, `rime/arcana:3-es`. See `voiceai voices --model rime/arcana:3-en` for the catalog.
- **Cartesia Sonic 3** — `cartesia/sonic:3`. WebSocket-only.
- **Sarvam Bulbul v3** — `sarvam/bulbul:3`. Hindi, Tamil, Telugu, Kannada, Marathi.
- **Kugel** — `kugelaudio/kugel:1`, `kugelaudio/kugel:2`, `kugelaudio/kugel:1-turbo`. WebSocket-only.
- **Murf Falcon** — `murf/falcon:1`. WebSocket-only.
- **Soniox TTS v1** — `soniox/tts:1`.

## Models by region

Some models are deployed only in certain regions. The CLI auto-routes; explicit override:

```bash
voiceai tts "..." --region eu-north-1
voiceai tts "..." --world-part eu
```

Available regions: `ap-south-1`, `ap-southeast-2`, `asia-south1`, `asia-southeast2`, `australia-southeast1`, `eu-north-1`, `us-east-1`.

Full per-region availability: https://docs.slng.ai (see Models by Region).

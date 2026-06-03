---
name: livekit-migration
description: Migrate an existing LiveKit Agents (Python) project to SLNG hosted speech infrastructure (STT + TTS) via the livekit-plugins-slng plugin. Use when the user asks to "move my LiveKit agent to slng", "use slng for STT/TTS in LiveKit", "swap deepgram/cartesia for slng", "add the slng livekit plugin", or "migrate livekit to slng".
license: MIT
compatibility: Requires a LiveKit Agents Python project (uv), git, internet access, and a SLNG_API_KEY.
---

# Migrate a LiveKit agent to slng

Move an existing [LiveKit Agents](https://docs.livekit.io/agents/) Python project onto slng hosted
speech infrastructure — speech-to-text and text-to-speech served through the
[`livekit-plugins-slng`](https://docs.slng.ai/agents/livekit-plugin) plugin. The plugin keeps the
rest of the agent (LLM, VAD, turn detection, room wiring) untouched, so this is a pipeline swap, not
a rewrite.

> **Scope:** the slng LiveKit plugin provides `slng.STT` and `slng.TTS` only. It does **not** ship an
> LLM class — your agent keeps its current LLM (LiveKit Inference, OpenAI, etc.). See step 6.

> **Setup:** the plugin authenticates with `SLNG_API_KEY`. If you do not have a key yet, the
> [`setup-api-key`](../setup-api-key/SKILL.md) skill walks through getting one from the
> [dashboard](https://slng.ai/dashboard/api-keys). Export the key value as `SLNG_API_KEY` so the
> plugin can read it (it reads that env var by default, or accepts `api_key=`).

This skill is **guided and reversible**. It does not edit blindly: it requires a clean git tree,
proves the plugin imports before touching code, makes idempotent edits, verifies with a test and a
boot check, and tells you exactly how to roll back. Follow the steps in order. **Do not skip the
gates in steps 1 and 2** — they are what make the migration safe.

The full model/voice/region catalog and the copy-pasteable wire-up live in
[`references/pipeline-reference.md`](references/pipeline-reference.md). The test template and
verification commands live in [`references/verification.md`](references/verification.md).

## 1. Safety gate — run before anything mutates

Confirm this is a LiveKit Agents project, and that it is a git repo of its own:

```bash
test -f src/agent.py && grep -q 'livekit-agents' pyproject.toml && echo "ok: livekit agents project"
git rev-parse --show-toplevel    # confirm the project root is itself a repo; `git init` it if not
```

Require a **clean git tree** so rollback is unambiguous:

```bash
git status --short      # must be empty
git diff --check        # must be clean (no whitespace errors / conflict markers)
```

If there are uncommitted changes, **stop** and ask the user to commit or stash first. If the project
is not its own git repo (e.g. it is an untracked subfolder of another repo), `git init` it and commit
the current state before migrating, so there is a real rollback point.

Confirm the API key is present **and valid**. Presence is not enough — an invalid key passes this
check and then fails much later as a cryptic `401` on the websocket handshake, mid-session. Never
echo, log, or paste the key value:

```bash
[ -n "$SLNG_API_KEY" ] && echo "SLNG_API_KEY is set" || echo "SLNG_API_KEY is NOT set"
# Validate it (200 = good, 401 = invalid/wrong workspace):
curl -s -o /dev/null -w "key check: HTTP %{http_code}\n" \
  -H "Authorization: Bearer $SLNG_API_KEY" https://api.agents.slng.ai/v1/agents
```

If it is missing or returns `401`, send the user to [`setup-api-key`](../setup-api-key/SKILL.md) to get
a valid key. **Store it in `.env.local`**, not just a shell export: the starter calls
`load_dotenv(".env.local")`, so a shell-only export works for `console` but the key goes missing under
`dev` and in deployment. Confirm `.env.local` is gitignored (it matches `.env.*` in the starter).

## 2. Install and probe the plugin — BLOCKING gate

```bash
uv add livekit-plugins-slng
uv run python -c "from livekit.plugins import slng; assert slng.STT and slng.TTS"
```

If the import **fails**, **stop before editing any code**. Do not guess an alternate import path or
package name. Report the failure to the user and point them at
<https://docs.slng.ai/agents/livekit-plugin> to confirm the current package and import surface. A
failed probe here means the migration cannot proceed reliably.

## 3. Detect the current pipeline

Read `src/agent.py` and report what is currently wired, so the swap is targeted. Note **where** each
stage lives — the placement differs by starter:

- **STT and TTS** are usually on the `AgentSession(stt=..., tts=...)`. These are what slng replaces.
- **LLM** is usually on the `Agent` subclass: `Agent.__init__(llm=...)`. The slng plugin has no LLM,
  so this stays as-is (see step 6).
- **VAD** (`silero`), **turn detection** (`MultilingualModel`), and any **noise cancellation** plugin
  (e.g. `ai_coustics`) are separate and are kept as-is unless the user asks otherwise.

Report the exact current STT/TTS models (e.g. `deepgram/nova-3`, `cartesia/sonic-3`) before changing them.

## 4. Ask only what cannot be inferred

Do not interrogate the user on every value. The default faithful migration routes the **same provider**
through slng — keep what the project already uses (e.g. `deepgram/nova-3` → `deepgram/nova:3`,
`cartesia/sonic-3` → `cartesia/sonic:3`, keeping the voice). Ask only when a value cannot be inferred or
the default is clearly not acceptable (an ambiguous region, or a voice the user has signalled they care
about). When you do ask, offer the default as the recommended option.

| Choice | Default (recommended) |
|--------|-----------------------|
| STT model | Current provider via slng, e.g. `deepgram/nova:3` (keep today's language) |
| TTS model + voice | Current provider via slng, e.g. `cartesia/sonic:3` (keep the voice) |
| Region | Auto (closest). Pin with `region_override`, e.g. `eu-north-1` for EU residency |

See [`references/pipeline-reference.md`](references/pipeline-reference.md) for the full catalog of
models, voices, and regions if the user wants something other than a like-for-like swap.

## 5. Apply the migration — keep it idempotent

Replace the STT and TTS constructors with the slng plugin classes. The key is read from
`SLNG_API_KEY` automatically; pass `api_key=os.environ["SLNG_API_KEY"]` explicitly if you want it
visible at the call site — **never a literal key string**.

- Add `from livekit.plugins import slng` to the imports **only if not already present**. Do not
  duplicate the import line.
- Replace STT/TTS where they live (often `AgentSession`). Leave the LLM, `vad`, `turn_detection`, and
  `preemptive_generation` exactly as they are.
- Optionally pass `region_override=...` per the chosen region.
- Do not add the `livekit-plugins-slng` dependency to `pyproject.toml` twice — `uv add` in step 2
  already recorded it.

**Idempotency check:** running this skill again on an already-migrated project must be a no-op. Before
inserting anything, confirm it is not already there. The exact before/after for the standard starter
is in [`references/pipeline-reference.md`](references/pipeline-reference.md).

Format and lint:

```bash
uv run ruff format
uv run ruff check
```

## 6. Subset and LLM handling — never leave it half-broken

The slng LiveKit plugin covers **STT and TTS only**. The agent's **LLM stays on its current
provider** — this is expected, not a failure. Tell the user the LLM was left as-is and on which
provider.

If a chosen STT or TTS model is unavailable, migrate the stage that works, leave the other on its
current provider so the agent still runs, and **clearly report which stage remains and how to finish
it**. Do not silently skip a stage and do not leave the project in a non-booting state.

## 7. Verify — and remember green tests are not a working agent

The unit test and `ruff` only prove the code is *wired* to the right strings. They pass even when the
agent is broken at runtime (a model that is unavailable, an invalid key, a provider that rejects the
request). **The only real acceptance gate is a live spoken turn** that completes STT → LLM → TTS.
Full commands and the test template are in [`references/verification.md`](references/verification.md).

Static checks first:

```bash
git diff --check                                              # tree is still clean of errors
git diff | grep -nE 'SLNG_API_KEY|api_key' ; echo "review ^"  # only os.environ[...], never a literal key
uv run pytest                                                 # asserts STT/TTS are slng.* with the right models
uv run python src/agent.py download-files
```

Then a **live turn**. Note how the agent is dispatched (it changes how you get audio in):

- `uv run python src/agent.py console` — local loopback, fastest way to speak to the agent in the
  terminal. Best for a quick STT/TTS check.
- `uv run python src/agent.py dev` — registers a worker with LiveKit Cloud and populates the Cloud
  observability panel. If the entrypoint is decorated with an `agent_name=` (explicit dispatch), the
  agent will **not** auto-join rooms — dispatch it (`lk dispatch create --new-room --agent-name <name>`)
  and join that room from a mic-enabled client (e.g. the [Agents Playground](https://agents-playground.livekit.io)),
  or it will sit idle and the panel will stay empty.

Both need a real `SLNG_API_KEY` and LiveKit credentials; the unit test runs with a dummy key. **Check
each stage independently** — STT can transcribe while TTS fails (or vice-versa). Confirm you see the
user transcript *and* hear the reply before declaring success. If a stage fails at runtime, see
Troubleshooting below and treat it as the step 6 subset case.

## 8. Roll back if needed

Because step 1 required a clean tree, rollback is clean:

```bash
git restore .            # discard the migration before committing
# or, after committing:
git revert <sha>
```

Suggested commit message once verified:

```
feat(voice): route STT/TTS through slng (livekit-plugins-slng)
```

## 9. Troubleshooting — runtime errors and what they mean

These surface only on a live turn, not in unit tests. Map the symptom to the cause before assuming a
code bug:

| Symptom (in worker logs) | Cause | Fix |
|--------------------------|-------|-----|
| `401` on `wss://api.slng.ai/...` handshake; `STT/TTS fallback exhausted` | Missing or invalid `SLNG_API_KEY` in the agent's environment | Put a **valid** key in `.env.local` (step 1). Validate with the `curl … /v1/agents` → `200` check |
| `400 Bad Request - Invalid Cartesia-Version header` | **Server-side gateway issue** — the [Cartesia Sonic 3 API](https://docs.slng.ai/api-reference/tts/cartesia-sonic-3/cartesia-sonic-3-ws) requires no client version header, so this is SLNG's to fix. Cartesia **is** supported; do **not** switch models because of it | Retry; report to SLNG via `lk docs submit-feedback`. Only as a temporary unblock, try another TTS model |
| No transcription **and** empty Cloud "Agent configuration" panel | The session never ran a healthy turn (usually the `401` above), or `dev` worker idle because an `agent_name=` agent was never dispatched | Fix the key; for explicit dispatch, dispatch the agent and join with a mic client (step 7) |
| `STT works but TTS fails` (or vice-versa) | A single stage's model is unavailable/broken | This is the step 6 subset case — swap that stage's model, keep the other |
| `ValueError: api_key is required` at construction | No key resolved when the component is built | Same as `401`: set `SLNG_API_KEY` before launch |

## 10. Report and submit docs feedback

Summarize what changed (STT/TTS models, region) and confirm the LLM was left on its current provider.
Per this project's `AGENTS.md`, file `lk docs submit-feedback` for any gap or inaccuracy you hit in
the slng or LiveKit docs during the migration — for example the package/import surface drifting from
the docs, or a server-side gateway error (like the Cartesia-Version `400`) that the client cannot fix.

## See also

- [`references/pipeline-reference.md`](references/pipeline-reference.md) — model/voice/region catalog and the exact wire-up block
- [`references/verification.md`](references/verification.md) — test template, git checks, boot/verify, rollback, acceptance checklist
- [`setup-api-key`](../setup-api-key/SKILL.md) — obtain and configure the API key

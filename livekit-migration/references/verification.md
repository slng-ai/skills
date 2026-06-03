# Verifying the migration

Run these after applying the migration. They are ordered cheapest-first: static checks, then the
unit test, then a live boot. All must pass before you consider the migration done.

## 1. Git safety checks

The tree should contain only the intended pipeline edits, with no whitespace or conflict errors:

```bash
git diff --check          # no whitespace errors or conflict markers
git status --short        # review exactly which files changed (agent.py, pyproject.toml, tests)
```

## 2. No-secret check

The diff must reference the key only through `os.environ`, never as a literal:

```bash
git diff | grep -nE 'api_key|SLNG_API_KEY'
```

Every match must be `api_key=os.environ["SLNG_API_KEY"]` (or absent, since the plugin reads the env
var automatically). If any line contains a quoted key value, remove it.

## 3. Unit test — STT/TTS resolve to slng and the configured models

Assert that the STT and TTS stages are slng instances wired to the intended models. This is the
regression guard against a future edit silently changing a provider. Add it to the existing `tests/`
directory.

The slng plugin stores the configured model id on private fields (`STT._model`, `TTS._opts.model`);
the public `.model` returns the provider name (`"slng"`), so assert on the stored id.

```python
# tests/test_slng_pipeline.py
import pytest
from livekit.plugins import slng


@pytest.fixture(autouse=True)
def _fake_key(monkeypatch):
    # The plugin reads SLNG_API_KEY; a dummy value is fine for a config assertion.
    monkeypatch.setenv("SLNG_API_KEY", "test-key")


def test_stt_is_slng_nova_3():
    stt = slng.STT(model="deepgram/nova:3", language="en")
    assert isinstance(stt, slng.STT)
    assert stt._model == "deepgram/nova:3"


def test_tts_is_slng_aura_2():
    tts = slng.TTS(model="slng/deepgram/aura:2-en", voice="aura-2-thalia-en")
    assert isinstance(tts, slng.TTS)
    assert tts._opts.model == "slng/deepgram/aura:2-en"
```

For a stronger test that asserts the models actually wired into `agent.py` (not just freshly
constructed objects), refactor the pipeline construction in `agent.py` into a small factory function
and assert on its output — this keeps the test honest if someone edits `agent.py` later. Private
fields can change across plugin versions; if `_model` / `_opts.model` move, re-introspect with
`uv run python -c "from livekit.plugins import slng; print(dir(slng.STT(model='deepgram/nova:3', api_key='x')))"`.

Run:

```bash
uv run pytest
```

## 4. Live-turn check — the real acceptance gate

A passing unit test and clean `ruff` only prove the code is wired to the right strings. They pass even
when the agent is broken at runtime (unavailable model, invalid key, provider rejecting the request).
**Only a live spoken turn that completes STT → LLM → TTS proves the migration works.** This step needs
a **real** `SLNG_API_KEY` (in `.env.local`, validated against `/v1/agents` → `200`) and LiveKit
credentials.

```bash
uv run python src/agent.py download-files
uv run python src/agent.py console        # local loopback; talk to it in the terminal; Ctrl-C to exit
```

For LiveKit Cloud observability, use `dev` instead. If the entrypoint uses `agent_name=` (explicit
dispatch), the worker will not auto-join rooms — dispatch it and join with a mic client:

```bash
uv run python src/agent.py dev
# in another shell:
lk dispatch create --new-room --agent-name <agent_name>
# then join that room from a mic-enabled client (e.g. https://agents-playground.livekit.io)
```

**Verify each stage independently** — STT can transcribe while TTS fails, or vice-versa. Confirm you
see the user transcript in the logs *and* hear the spoken reply. If one stage fails at runtime, see
the Troubleshooting table in `SKILL.md` and handle it as the subset case. Check first-token latency in
the [slng dashboard](https://slng.ai/dashboard).

## 5. Lint

```bash
uv run ruff format
uv run ruff check
```

## Acceptance checklist

- [ ] `git status` was clean before the migration started (clean rollback point).
- [ ] `from livekit.plugins import slng` imports successfully.
- [ ] STT and TTS are `slng.STT` / `slng.TTS` wired to the intended models.
- [ ] The LLM was left on its current provider and the user was told so.
- [ ] No API key literal appears in the diff — only `os.environ[...]` (or the env var alone).
- [ ] `uv run pytest` passes, including the STT/TTS assertions.
- [ ] `SLNG_API_KEY` is in `.env.local` and validates against `/v1/agents` → `200`.
- [ ] A **live turn** completes: user transcript appears in logs **and** the spoken reply is heard (each stage verified independently).
- [ ] Any stage left on its original provider is reported to the user with how to finish it.
- [ ] `agent.py` is still the entrypoint and `uv run ruff check` is clean.

## Rollback

Because the migration started from a clean tree:

```bash
git restore .             # before committing: discard all migration edits
git checkout -- .         # equivalent
git revert <sha>          # after committing: reverse the migration commit
```

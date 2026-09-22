---
name: python-starter
description: Scaffolds a Python + pytest automation project following principal-automation-engineer standards. Load this when the user says "start a new Python test project", "scaffold a pytest project", "create an automation framework", or any request to build a new Python testing codebase from scratch. Use ONLY for greenfield Python test/automation projects, not for adding tests to an existing project.
---

You are a Principal Automation Engineer scaffolding a new Python + pytest project.

## Step 0 — Gather requirements first (guided)

Before scaffolding, ask the user these questions in order. Items marked *recommend* ship with a default — present it plus its one-line reason; the user's answer always wins. Record answers and overrides in the README's design-decisions note.

1. **Assignment** — What system are you testing? Pick the closest match:
   - REST API / gRPC service
   - UI (web / desktop)
   - CLI
   - Library / SDK
   - **Distributed component** — job queue, batch processor, object/file store, message consumer, async API client

   *Recommend: if it's a distributed component, complete Q6 (reliability scope) too.*

2. **System under test** — What's the base URL / app path / package? Service you control (spawnable), external (fixed URL), or a simulated/toy component you'll also build in-repo?

3. **Tech stack** — Allowed libraries? (requests / httpx / playwright / selenium / asyncio / etc.)

   *Recommend: mirror what the SUT uses — sync `requests` for sync HTTP, `httpx` + `pytest-asyncio` for async, `playwright` for browsers. Never add a library the SUT doesn't already depend on.*

4. **Constraints** — Time limit, must run offline, no network, CI-only, stdlib-only?

5. **Package name** — Name for the Python package (default: derived from project dir).

6. **Reliability scope** — Does the assignment need reliability coverage? Multi-select, or answer "none":
   - Timeouts + retries with **exponential backoff**
   - **Idempotency** guarantees (retry-safe operations; state effect happens exactly once)
   - **Failure modes** — simulated latency, packet loss, node disconnects, race conditions
   - **Async** flows (`pytest-asyncio`, async client), if the SUT is async
   - **State verification** across components (N-in/N-out, zero duplicates, read-your-writes)
   - None — core scaffold only

   *Recommend (minimum set): "timeouts + retry/backoff" — highest reliability value for near-zero cost. Add idempotency + state verification whenever a request can be replayed (queues, batches, stores). Include failure modes when the distributed row of the strategy matrix applies.*

7. **Fault-injection style** — *only asked if "failure modes" was selected in Q6*:
   - **Deterministic scheduler** at the SUT boundary — fake clock + scripted faults, zero infrastructure — *recommended first*
   - **Real multi-node topology** — Docker Compose + chaos proxy (TCP-level drops, kill/restart nodes) — *phase 2, see `references/topology.md`*

   *Recommend: start deterministic; graduate to real topology only when a fake can't express the failure (half-open TCP, kill -9 mid-ack).*

**Module routing** — before writing any code, read the files matching the Q6/Q7 answers:

| Selected in Step 0 | Read first |
|---|---|
| Nothing reliability-related | core SKILL.md only |
| Timeouts / retries / idempotency / state / async | `references/reliability-patterns.md` |
| Failure modes — deterministic | `references/reliability-patterns.md` (FaultScheduler section) |
| Failure modes — multi-node / chaos | `references/topology.md`, after reliability-patterns |
| Any reliability option | `references/strategy-matrix.md` — fill the distributed-failure row |

If the user is preparing to demonstrate these skills (interview, review, rubric), share `references/concepts-to-review.md` first — it maps each expected concept to where it appears in the scaffold.

Adapt the generated code to these answers. Do not hardcode assumptions.

## Step 1 — Environment model (APP_ENV)

Environments are selected by the `APP_ENV` env var (default `dev`), never by editing YAML profiles. No `config.yaml`. Credentials are never registry keys.

```
project_root/
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── clients/                       # thin API/UI adapters (HTTP calls live here)
│       ├── services/                      # workflows that orchestrate clients
│       └── models/                        # dataclasses for request/response data
├── tests/
│   ├── conftest.py                        # APP_ENV registry (topology only), server strategies, secret loading, prod guard
│   ├── data/                              # test-owned data: sample payloads, expected JSON (never credentials)
│   ├── unit/
│   │   ├── conftest.py                    # auto-marks every test in this dir as unit
│   │   └── test_*.py
│   ├── integration/
│   │   ├── conftest.py                    # auto-marks every test in this dir as integration
│   │   └── test_*.py
│   └── e2e/
│       ├── conftest.py                    # auto-marks every test in this dir as e2e
│       └── test_*.py
├── pyproject.toml                         # pytest config (testpaths, markers) + test deps
├── requirements.txt                       # pinned versions (pytest-mock, pytest-check, python-dotenv)
├── README.md                              # setup, run commands, design notes
├── .env.example                           # committed secret-key template — keys only, no values
└── .gitignore                             # .env, __pycache__, .pytest_cache
```

**The conftest registry** — `TEST_CONFIG` maps each env to topology only (API URL, server strategy, ports, node ids). No credentials. Server strategies: `testclient` (in-process ASGI, default safe), `uvicorn` (real subprocess, xdist port offsets), `container` (`docker compose`, requires Docker), `local` (assume a running server).

**Safety rules enforced in `tests/conftest.py`** (import-time):
- `APP_ENV` defaults to `dev`; any value not in `TEST_CONFIG` → `pytest.exit`.
- `APP_ENV=prod` requires `--run-prod`.
- Secrets loaded explicitly at import from a gitignored `.env` (or CI env vars) via `python-dotenv` into a `SECRETS` mapping, then stripped from `os.environ`.

**Key rule**: Keep test logic out of `src/` and business logic out of `tests/`.

## Step 2 — Coding guidelines

- **Ask before assuming**: always begin by asking the assignment questions above before writing any code.
- **Clear naming**: snake_case for files/functions/variables, PascalCase for classes. Verb-first functions (`create_order`, `resolve_env`). Test names: `test_<behavior>_<expected_outcome>`.
- **Right abstraction**: separate clients -> services -> tests. Wrap external calls in a client class so tests can mock at that boundary. Builder/Factory patterns are encouraged for test data.
- **Clean OOP**: small classes with a single responsibility; prefer composition over inheritance; no god objects.
- **Error handling**: `try/except` around I/O and network calls; catch specific exceptions, never bare `except:`. Raise clear domain exceptions with context. Add timeouts to every network call. Fail fast on bad config.
- **Polling & timeout conditions**: bounded polling loop with an overall deadline (`while time.monotonic() < deadline`); never `while True` without a cap, never unbounded `time.sleep`.
- **Debuggable failures**: every failure must be actionable — log context (request, params, response).
- **Readable over clever**: small functions, early returns, type hints on public functions, dataclasses instead of loose dicts.
- **Security**: no hardcoded secrets/URLs — non-secret settings from the `TEST_CONFIG` registry, credentials from `.env`/CI env vars only. Never log credentials or PII. No `shell=True`.
- **Mocking**: use `pytest-mock` (`mocker` fixture) for all mocking — never use `responses` or other third-party mock libraries. `mocker` auto-reverts every patch after the test. Two rules:
  - **Mock at the boundary, fake real behavior.** Stateful collaborators (stores, counters, node instances) stay real objects so state assertions test real logic. Only the SUT's genuine seam gets patched: the HTTP request function (with a `_resp()` helper) or, for in-process/library SUTs, the method call — `mocker.patch.object(node, "write_chunk", ...)` — never the whole class.
  - **Faults are scripted, not random.** Inject faults via `side_effect` sequences that fall through to the real implementation, plus an injectable `FakeClock` for backoff. The SUT itself carries no chaos logic (no `failure_rate`, no seeded `random`) — determinism always comes from the tests.
- **Pytest discipline**: Arrange-Act-Assert; fixtures in `conftest.py`; `@pytest.mark.parametrize` for data cases; markers `unit`/`integration`/`e2e` via auto-marking nested conftests; tests must be independent and order-free. Cover happy path + validation failure + one edge/error case per critical flow.
- **Complex response validation (soft assertions)**: when a single response has 10+ fields, use `pytest-check` soft assertions, every `check.*` carrying `msg="<field>"`. Soft asserts report, they don't halt — flow-critical preconditions stay hard `assert`s.
- **Comments**: explain *why*, not *what*. No narration comments.

## Step 3 — Build order

1. `pyproject.toml` + `requirements.txt` (pytest-mock, pytest-check, python-dotenv)
2. `.env.example` + `tests/conftest.py` — `TEST_CONFIG` registry, `_resolve_env`, secret loading/stripping, `--run-prod` guard, client fixtures
3. One client (the thin adapter for the system under test)
4. Models for the data exchanged
5. Nested conftests (auto-markers) + tests (unit first, then integration/e2e) + any `tests/data/` files
6. If reliability scope was selected (Q6): `tests/faults.py` (FaultScheduler + FakeClock for HTTP SUTs; `scripted_writes` / `down_writes` mocker helpers for in-process SUTs) + reliability tests + fill the coverage matrix
7. `README.md` with how to install, configure (`APP_ENV`), and run

## Step 4 — Working rules

- **Start with Step 0**: ask the user for their assignment before doing anything else.
- Generate **one file at a time**, smallest correct version. Pause for user review before the next file.
- After meaningful changes, show the exact `pytest` command to run and expected outcome.
- If unsure about a requirement, ask **one** focused question instead of guessing.
- Keep total dependencies minimal and justify any third-party library.
- Prefer finishing a small, fully-working slice over a large half-working one.

## Definition of done

- [ ] Structure matches the layout above (`pyproject.toml`, not `pytest.ini`)
- [ ] `pytest` passes with documented commands; env selected via `APP_ENV`, defaults to `dev`
- [ ] No secrets/URLs hardcoded; `TEST_CONFIG` holds topology only, `.env.example` committed, prod guard + secret stripping present
- [ ] Clear names, type hints, specific exception handling
- [ ] Complex-response tests validate all fields via `pytest-check` soft assertions
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note (tradeoffs + what I'd add in production)
- [ ] If reliability scope was selected (Q6): at least one deterministic fault-injection test, one retry-with-backoff test using an injectable clock (no real sleeps), and one idempotency/state-verification test — all mapped in the coverage matrix
---
name: python-test-starter
description: |
  Scaffolds a Python + pytest automation project following principal-automation-engineer standards.
  Load this when the user says "start a new Python test project", "scaffold a pytest project",
  "create an automation framework", or any request to build a new Python testing codebase from scratch.
  Use ONLY for greenfield Python test/automation projects, not for adding tests to an existing project.
---

# Python + pytest Test Starter

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
   - **Deterministic scheduler** at the HTTP-client boundary — fake clock + scripted faults, zero infrastructure — *recommended first*
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

Environments are selected by the `APP_ENV` env var, never by editing YAML profiles. No `config.yaml`. Credentials are never registry keys — see "Registry vs secrets (hybrid)" below.

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
├── requirements.txt                       # pinned versions (include python-dotenv for secrets)
├── README.md                              # setup, run commands, design notes
├── .env.example                           # committed secret-key template — keys only, no values
└── .gitignore                             # .env, __pycache__, .pytest_cache
```

**The conftest registry** — mirror the `TEST_CONFIG` pattern from the parkinglot repo. Each env maps to an API URL and a server strategy:

```python
TEST_CONFIG = {
    "dev":       {"api_url": "http://testserver",      "server": "testclient"},
    "qa":        {"api_url": "http://localhost:9999",  "server": "uvicorn"},
    "staging":   {"api_url": "http://localhost:9998",  "server": "uvicorn"},
    "e2e":       {"api_url": "http://localhost:8000",  "server": "local"},
    "prod":      {"api_url": "http://testserver",      "server": "testclient"},
    "local":     {"api_url": "http://localhost:8000",  "server": "local"},
    "ci":        {"api_url": "http://localhost:8003",  "server": "container"},
}

def _resolve_env() -> str:
    env = os.getenv("APP_ENV", "dev")
    if env not in TEST_CONFIG:
        pytest.exit(f"Invalid APP_ENV: {env}")
    return env
```

**Server strategies** (from parkinglot `tests/conftest.py`):

- `testclient` — in-process ASGI client against the app import; use a temp DB per fixture and clean up after. Fastest, default for `dev`/`prod` safety.
- `uvicorn` — spawn a real `uvicorn` subprocess on the configured port (with xdist worker port offsets); wait for readiness before yielding.
- `container` — `docker compose up -d --build` then `down -v`; require Docker or `pytest.exit`.
- `local` — assume a server is already running at `api_url` (e.g. a dev box or external env); just point the client at it.

Always use temp/sandboxed resources for anything you spawn (temp DB file per fixture, `JWT_SECRET=test-secret`, etc.) and clean up in `finally`.

**Safety rules enforced in `tests/conftest.py`** (import-time):

- `APP_ENV` defaults to `dev`; any value not in `TEST_CONFIG` → `pytest.exit`.
- `APP_ENV=prod` requires `--run-prod` (added via `parser.addoption` in `pytest_addoption`); `pytest_configure` aborts otherwise.
- Secrets are loaded explicitly at import from a gitignored `.env` (or CI env vars) via `python-dotenv` into a `SECRETS` mapping, then stripped from `os.environ` (`os.environ.pop`) so client code can never pick them up accidentally. Deliberate loading is the intended path; stripping is only the safety net. Missing secrets fail fast at fixture time with a pointer to `.env.example`.

**Registry vs secrets (hybrid)**: `TEST_CONFIG` holds **only non-secret, static topology** (API URL, server strategy, ports). Credentials of any kind — including demo/test creds — are secrets and are **never valid registry keys**. All credentials live in a gitignored `.env` (committed template: `.env.example`) or CI env vars, loaded via `python-dotenv` at import and stripped from the environment after capture. Treating even demo creds as secrets keeps the boundary binary: no judgment call in review about what counts as a secret.

**Config vs test data**: There is no `tests/config/` that re-declares runtime settings — that causes drift. Non-secret runtime settings (API URL, server strategy) come from the `TEST_CONFIG` registry via `APP_ENV`. Test-owned artifacts (sample payloads, expected responses) live in `tests/data/` — but credentials never do: they are secrets and live in `.env`/CI env only.

**Key rule**: Keep test logic out of `src/` and business logic out of `tests/`.

## Step 2 — Coding guidelines

- **Ask before assuming**: Always begin by asking what the user is testing. Gather SUT, tech stack, and constraints before writing any code.

- **Clear naming**: snake_case for files/functions/variables, PascalCase for classes. Verb-first functions (`create_order`, `resolve_env`). No cryptic abbreviations, no 8-word names. Test names: `test_<behavior>_<expected_outcome>`.
- **Right abstraction**: separate config -> clients -> services -> tests. Wrap external calls in a client class so tests can mock at that boundary. No factories-of-factories, custom DSLs, or base classes "just in case." Builder/Factory patterns **are** encouraged for test data — see `references/concepts-to-review.md`.

- **Clean OOP**: small classes with a single responsibility; prefer composition over inheritance; no god objects. Each client/service should do one job and be independently testable.
- **Error handling**: `try/except` around I/O and network calls; catch specific exceptions, never bare `except:`. Raise clear domain exceptions with context. Add timeouts to every network call. Fail fast on bad config (`pytest.exit` on invalid `APP_ENV`). Defensive: validate inputs; never silently swallow a failure.

- **Polling & timeout conditions**: for readiness checks or state waits, use a bounded polling loop with an overall deadline — `while time.monotonic() < deadline` with a short interval, then fail with context. Never `while True` without a cap; never unbounded `time.sleep`.

- **Debuggable failures**: every failure must be actionable — log context (request, params, response) so a red test tells you what broke and where, not just that it broke. Prefer a clear assertion message over re-running to debug.
- **Readable over clever**: small functions (one job each), early returns over deep nesting, type hints on public functions, dataclasses instead of loose dicts. If a line needs a comment to be understood, simplify it.
- **Security**: no hardcoded secrets/URLs — base URLs read from the `TEST_CONFIG` registry; credentials read from `.env`/CI env vars (never the registry, never source). Never log credentials or PII. No `shell=True`. Secrets are captured explicitly at import, then stripped from the environment.
- **Mocking**: Use `pytest-mock` (`mocker` fixture) for all mocking — never use `responses` or other third-party mock libraries. `mocker` auto-reverts every patch after the test, so patches can't leak between tests. Two rules:
  - **Mock at the boundary, fake real behavior.** Stateful collaborators (stores, counters, node/service instances) stay **real objects** so state assertions test real logic. Only the SUT's genuine seam gets patched: for HTTP that's the request function (`requests.request`/your lib's request fn) with a `_resp()` helper building mock Response objects; for an in-process/library SUT (the distributed-component assignment) it's the method call — e.g. `mocker.patch.object(node, "write_chunk", ...)` — **never the whole class**.
  - **Faults are scripted, not random.** Inject faults via `side_effect` sequences that fall through to the real implementation, plus an injectable `FakeClock` for backoff. The SUT itself carries **no chaos logic** (no `failure_rate`, no seeded `random`) — determinism always comes from the tests. See `references/reliability-patterns.md` section 3.
- **Pytest discipline**: Arrange-Act-Assert; fixtures in `conftest.py`; `@pytest.mark.parametrize` for data cases; markers `unit`/`integration`/`e2e`; tests must be independent and order-free. Mock HTTP/external systems, not internal logic. Cover happy path + validation failure + one edge/error case per critical flow. Each test dir auto-marks itself via `pytest_collection_modifyitems` comparing `Path(item.fspath).parent` to the conftest's own directory — no manual markers.
- **Complex response validation (soft assertions)**: When a single response has 10+ fields to validate (e.g. a full user profile), use soft assertions so one bad field doesn't hide the rest. Add `pytest-check` to `requirements.txt` and validate every field with `msg=` naming the field:

  ```python
  from pytest_check import check

  def test_profile_shape(profile) -> None:
      check.equal(profile["username"], "emilys", msg="username")
      check.is_true(bool(profile["email"]), msg="email is non-empty")
      check.greater(profile["id"], 0, msg="id is positive")
      check.is_instance(profile["role"], str, msg="role is a string")
  ```

  Rules:
  - Every `check.*` call carries `msg="<field>"` (plus expected/actual where useful) — a red test then lists each failing field at once (`Failed Checks: N`), not just the first one.
  - Soft asserts **report, they don't halt**: never gate a later step on a soft check — flow-critical preconditions (e.g. login succeeded, page loaded) stay hard `assert`s. Reserve soft assertions for bulk field/schema validation.
  - Keep hard `assert` for one-or-two field checks; soft assertions are for bulk validation.
  - `check.raises(Expected)` covers exception cases. Do not use `check.msg()` — it is not callable in this API; use the `msg=` keyword instead.
- **Comments**: explain *why*, not *what*. No narration comments.

## Step 3 — Build order

1. `pyproject.toml` (`[tool.pytest.ini_options]` with `pythonpath`, `testpaths`, `markers`, `log_cli`) + `requirements.txt` (include `pytest-mock` for boundary mocking, `pytest-check` for soft assertions and `python-dotenv` for secrets)
2. `.env.example` (secret-key template) + `tests/conftest.py` — `TEST_CONFIG` registry (topology only), `_resolve_env`, secret loading/stripping (python-dotenv), `--run-prod` guard, client fixtures (server strategies)
3. One client (the thin adapter for the system under test)
4. Models for the data exchanged
5. Nested conftests (auto-markers) + tests (unit first, then integration/e2e) + any `tests/data/` files
6. **If reliability scope was selected (Q6)**: `tests/faults.py` (`FaultScheduler` + `FakeClock` for HTTP SUTs; `scripted_writes` / `down_writes` mocker helpers for in-process SUTs) + reliability tests per `references/reliability-patterns.md` + fill the coverage rows per `references/strategy-matrix.md`
7. `README.md` with how to install, configure (`APP_ENV`), and run (`pytest`, `pytest -m unit`, `APP_ENV=prod pytest --run-prod`)

## Step 4 — Working rules

- **Start with Step 0**: Ask the user for their assignment before doing anything else.
- Generate **one file at a time**, smallest correct version. Pause for user review before the next file.
- After meaningful changes, show the exact `pytest` command to run and expected outcome.
- If unsure about a requirement, ask **one** focused question instead of guessing.
- Keep total dependencies minimal and justify any third-party library.
- Prefer finishing a small, fully-working slice over a large half-working one.

## Definition of done

- [ ] Structure matches the layout above (`pyproject.toml`, not `pytest.ini`)
- [ ] `pytest` passes with documented commands; env selected via `APP_ENV`, defaults to `dev`
- [ ] No secrets/URLs hardcoded; `TEST_CONFIG` holds topology only, credentials live in gitignored `.env`/CI env (`.env.example` committed), prod guard + secret stripping present
- [ ] Clear names, type hints, specific exception handling
- [ ] Complex-response tests validate all fields and aggregate failures via `pytest-check` soft assertions (not the first failing `assert`)
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note (tradeoffs + what I'd add in production)
- [ ] **If reliability scope was selected (Q6):** at least one deterministic fault-injection test (latency / drop / race), one retry-with-backoff test using an injectable clock (no real sleeps), and one idempotency/state-verification test — all mapped in the coverage matrix from `references/strategy-matrix.md`

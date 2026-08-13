---
name: python-test-starter
description: |
  Scaffolds a Python + pytest automation project following principal-automation-engineer standards.
  Load this when the user says "start a new Python test project", "scaffold a pytest project",
  "create an automation framework", or any request to build a new Python testing codebase from scratch.
  Use ONLY for greenfield Python test/automation projects, not for adding tests to an existing project.
---

# Python + pytest Test Starter

## Step 0 — Gather requirements first

Before scaffolding, ask the user these questions:

1. **Assignment** — What system are you testing? (REST API / UI / CLI / library)
2. **System under test** — What's the base URL / app path / package being tested? Is it a service you control (spawnable) or external (fixed URL)?
3. **Tech stack** — Allowed libraries? (requests / httpx / playwright / selenium / etc.)
4. **Constraints** — Time limit, must run offline, no network, etc.
5. **Package name** — Name for the Python package (default: derived from project dir)

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
- **Right abstraction**: separate config -> clients -> services -> tests. Wrap external calls in a client class so tests can mock at that boundary. No factories-of-factories, custom DSLs, or base classes "just in case."

- **Clean OOP**: small classes with a single responsibility; prefer composition over inheritance; no god objects. Each client/service should do one job and be independently testable.
- **Error handling**: `try/except` around I/O and network calls; catch specific exceptions, never bare `except:`. Raise clear domain exceptions with context. Add timeouts to every network call. Fail fast on bad config (`pytest.exit` on invalid `APP_ENV`). Defensive: validate inputs; never silently swallow a failure.

- **Debuggable failures**: every failure must be actionable — log context (request, params, response) so a red test tells you what broke and where, not just that it broke. Prefer a clear assertion message over re-running to debug.
- **Readable over clever**: small functions (one job each), early returns over deep nesting, type hints on public functions, dataclasses instead of loose dicts. If a line needs a comment to be understood, simplify it.
- **Security**: no hardcoded secrets/URLs — base URLs read from the `TEST_CONFIG` registry; credentials read from `.env`/CI env vars (never the registry, never source). Never log credentials or PII. No `shell=True`. Secrets are captured explicitly at import, then stripped from the environment.
- **Mocking**: Use `pytest-mock` (`mocker` fixture) for all mocking. Never use `responses` or other third-party mock libraries. Patch at the HTTP client boundary (`requests.request` / your HTTP lib's request function) with a helper like `_resp()` that builds mock Response objects. This keeps mocks explicit, debuggable, and avoids extra dependencies.
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

1. `pyproject.toml` (`[tool.pytest.ini_options]` with `pythonpath`, `testpaths`, `markers`, `log_cli`) + `requirements.txt` (include `pytest-check` for soft assertions and `python-dotenv` for secrets)
2. `.env.example` (secret-key template) + `tests/conftest.py` — `TEST_CONFIG` registry (topology only), `_resolve_env`, secret loading/stripping (python-dotenv), `--run-prod` guard, client fixtures (server strategies)
3. One client (the thin adapter for the system under test)
4. Models for the data exchanged
5. Nested conftests (auto-markers) + tests (unit first, then integration/e2e) + any `tests/data/` files
6. `README.md` with how to install, configure (`APP_ENV`), and run (`pytest`, `pytest -m unit`, `APP_ENV=prod pytest --run-prod`)

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
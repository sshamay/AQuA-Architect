---
name: python-playwright-starter
description: |
  Scaffolds a Python + pytest + Playwright UI automation project following principal-automation-engineer standards.
  Load this when the user says "start a new Python Playwright test project", "scaffold a Playwright pytest project",
  "create a Python automation framework", or any request to build a new Python UI testing codebase from scratch.
  Use ONLY for greenfield Python/Playwright UI test projects, not for adding tests to an existing project.
---

# Python + pytest + Playwright Starter

## Step 0 — Gather requirements first

Before scaffolding, ask the user these questions:

1. **Assignment** — What system are you testing? (UI only / UI + REST API)
2. **System under test** — What's the base URL / app path / package being tested? Is it an app you control (server can be spawned) or an external site (fixed URL)?
3. **Tech stack** — Allowed libraries? (`pytest-playwright` / `requests` / `httpx` / etc.)
4. **Constraints** — Time limit, must run offline, no network, browsers available, etc.
5. **Package name** — Name for the Python package (default: derived from project dir)

Adapt the generated code to these answers. Do not hardcode assumptions.

## Step 1 — Environment model (APP_ENV)

Environments are selected by the `APP_ENV` env var, never by editing YAML profiles. No `config.yaml`, no `.env`.

```
project_root/
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── pages/                         # Page Object Models, one class per screen
│       ├── fixtures/                      # pytest fixtures (logged-in session, page objects)
│       └── models/                        # dataclasses for exchanged data (address, card, product)
├── tests/
│   ├── conftest.py                        # APP_ENV registry, secret stripping, prod guard, shared fixtures
│   ├── data/                              # test-owned data: sample payloads, expected values
│   ├── e2e/
│   │   ├── conftest.py                    # auto-marks every test in this dir as e2e
│   │   └── test_*.py
│   └── api/                               # REST tests using requests/httpx (if applicable)
│       └── conftest.py                    # auto-marks every test in this dir as api
├── pyproject.toml                         # pytest config (testpaths, markers) + test deps
├── requirements.txt                       # pinned versions
├── README.md                              # setup, run commands (pytest -m e2e), design notes
└── .gitignore                             # secrets, __pycache__, .pytest_cache, test-results
```

**The conftest registry** — mirror the `TEST_CONFIG` pattern from the parkinglot repo. For a UI suite the registry maps each env to its `base_url` and credentials:

```python
TEST_CONFIG = {
    "dev":   {"base_url": "http://localhost:8000", "email": "dev@example.com", "password": "dev-pass"},
    "test":  {"base_url": "http://localhost:8001", "email": "test@example.com", "password": "test-pass"},
    "qa":    {"base_url": "http://qa.example.com",  "email": "qa@example.com",  "password": "qa-pass"},
    "e2e":   {"base_url": "http://e2e.example.com", "email": "e2e@example.com", "password": "e2e-pass"},
    "prod":  {"base_url": "http://example.com",     "email": "prod@example.com","password": "prod-pass"},
    "local": {"base_url": "http://localhost:8000",  "email": "dev@example.com", "password": "dev-pass"},
    "ci":    {"base_url": "http://localhost:8001",  "email": "test@example.com","password": "test-pass"},
}

def _resolve_env() -> str:
    env = os.getenv("APP_ENV", "dev")
    if env not in TEST_CONFIG:
        pytest.exit(f"Invalid APP_ENV: {env}")
    return env
```

**Safety rules enforced in `tests/conftest.py`** (import-time):

- `APP_ENV` defaults to `dev`; any value not in `TEST_CONFIG` → `pytest.exit`.
- `APP_ENV=prod` requires `--run-prod` (added via `parser.addoption` in `pytest_addoption`); `pytest_configure` aborts otherwise.
- Known secret env vars (`API_KEY`, `AUTH_TOKEN`, `DB_PASSWORD`) are stripped from the environment on import (`os.environ.pop`) so client code never picks them up.
- Credentials come from the `TEST_CONFIG` registry (or per-env env vars like `DREAM_EMAIL`/`DREAM_PASSWORD` with the registry defaulting to them), never hardcoded in tests.

**External site vs server you control**: if the SUT is a fixed external URL (e.g. a public demo site), the registry just holds `base_url` + creds and there is no server to spawn. If the SUT is a service you own, add the parkinglot server-strategy pattern (`testclient` / `uvicorn` / `container` / `local`) to the same registry and spawn/wait per fixture.

**Config vs test data**: There is no `tests/config/` that re-declares runtime settings — that causes drift. Runtime settings (base_url, credentials) come from the `TEST_CONFIG` registry / `APP_ENV`. Test-owned artifacts (sample payloads, expected values) live in `tests/data/`, never in the app config.

**Key rule**: Keep test logic out of `src/` and business logic out of `tests/`.

## Step 2 — Coding guidelines

- **Ask before assuming**: Always begin by asking what the user is testing. Gather SUT, tech stack, and constraints before writing any code.

- **Clear naming**: snake_case for files/functions/variables, PascalCase for classes. Verb-first functions (`fill_billing_address`, `resolve_env`). No cryptic abbreviations, no 8-word names. Test names: `test_<behavior>_<expected_outcome>`.

- **Right abstraction**: separate config -> pages -> tests. Page Objects wrap UI interaction; a typed client class wraps HTTP calls so tests can mock at that boundary. No factories-of-factories, custom DSLs, or base classes "just in case."

- **Clean OOP**: small classes with a single responsibility; prefer composition over inheritance; no god objects. Each Page Object should do one job and be independently testable. Encapsulation — Page Objects hide selectors and UI logic from tests.

- **Selectors**: prefer `page.get_by_role()` / `page.get_by_test_id()`, never brittle CSS strings buried in tests. Expose selectors via Page Objects, not inline in tests. Configure `test_id_attribute` for the app under test (e.g. `data-test`) in `tests/conftest.py` or `pytest-playwright` options.

- **Deterministic UI**: use Playwright's auto-waiting locators and `expect(locator).to_be_visible()` etc. from `pytest_playwright`/`expect`. Never `time.sleep()`/arbitrary sleeps. Isolated state per test: fresh context/page, no order dependence.

- **Error handling**: `try/except` around I/O and network calls; catch specific exceptions, never bare `except:`. Raise clear domain exceptions with context. Add timeouts to every network call. Fail fast on bad config (`pytest.exit` on invalid `APP_ENV`). Defensive: validate inputs; never silently swallow a failure.

- **Debuggable failures**: every failure must be actionable — log context (page state, URL) so a red test tells you what broke and where, not just that it broke. Prefer a clear assertion message over re-running to debug.

- **Security**: no hardcoded secrets/URLs — read from the `TEST_CONFIG` registry or per-env env vars, never in source. Never log credentials or PII. Secrets are stripped at import.

- **Mocking**: Use `page.route()` / `context.route()` to intercept API calls at the HTTP boundary. Mock external systems, not internal logic.

- **Pytest discipline**: Arrange-Act-Assert; shared fixtures in `tests/conftest.py` (or `src/<package>/fixtures/` for reusable app fixtures); `@pytest.mark.parametrize` for data cases; markers `e2e`/`api`; tests must be independent and order-free. Each test dir (`tests/e2e/`, `tests/api/`) auto-marks itself via `pytest_collection_modifyitems` comparing `Path(item.fspath).parent` to the conftest's own directory — no manual markers.

- **Type discipline**: type hints on all public signatures; dataclasses for exchanged data instead of loose dicts.

- **Comments**: explain *why*, not *what*. No narration comments.

## Step 3 — Build order

1. `pyproject.toml` (`[tool.pytest.ini_options]` with `pythonpath`, `testpaths`, `markers`, `log_cli`) + `requirements.txt`
2. `tests/conftest.py` — `TEST_CONFIG` registry, `_resolve_env`, secret stripping, `--run-prod` guard, base-url fixture, page-object fixtures
3. One Page Object for the system under test
4. `tests/e2e/conftest.py` (auto-marker) + first `tests/e2e/` test
5. `README.md` with how to install, configure (`APP_ENV`), and run (`pytest`, `pytest -m e2e`, `APP_ENV=prod pytest --run-prod`)

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
- [ ] No secrets/URLs hardcoded; registry/env-driven with prod guard and secret stripping
- [ ] Clear names, type hints, web-first assertions (no sleeps)
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note (tradeoffs + what I'd add in production)

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
2. **System under test** — What's the base URL / app path / package being tested?
3. **Tech stack** — Allowed libraries? (requests / httpx / playwright / selenium / etc.)
4. **Constraints** — Time limit, must run offline, no network, etc.
5. **Package name** — Name for the Python package (default: derived from project dir)

Adapt the generated code to these answers. Do not hardcode assumptions.

## Step 1 — Folder structure

```
project_root/
├── config/
│   ├── config.yaml                        # ONE source of truth: env profiles (dev/staging/test)
│   └── config.example.yaml                # committed template
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── config_loader.py               # load + validate YAML into typed settings
│       ├── logging_setup.py               # one place to configure logging
│       ├── clients/                       # thin API/UI adapters (HTTP calls live here)
│       ├── services/                      # workflows that orchestrate clients
│       └── models/                        # dataclasses for request/response data
├── tests/
│   ├── conftest.py                        # shared fixtures: loads config with env="test"
│   ├── data/                              # test-owned data: sample payloads, expected JSON
│   ├── unit/                              # fast, isolated tests with mocks
│   └── integration/                       # tests hitting real/stubbed dependencies
├── pytest.ini                             # markers, testpaths
├── requirements.txt                       # pinned versions
├── README.md                              # setup, run commands, design notes
└── .gitignore                             # secrets, __pycache__, .pytest_cache
```

**Key rule**: Keep test logic out of `src/` and business logic out of `tests/`.

**Config vs test data**: There is no `tests/config/` that re-declares runtime settings — that causes drift. Instead, the root `config.yaml` holds a `test` env profile, and `conftest.py` loads it via the same `config_loader` (`load_config(env="test")`). Test-owned artifacts (sample payloads, expected responses) live in `tests/data/`, never in the app config.

## Step 2 — Coding guidelines

- **Ask before assuming**: Always begin by asking what the user is testing. Gather SUT, tech stack, and constraints before writing any code.

- **Clear naming**: snake_case for files/functions/variables, PascalCase for classes. Verb-first functions (`create_order`, `load_config`). No cryptic abbreviations, no 8-word names. Test names: `test_<behavior>_<expected_outcome>`.
- **Right abstraction**: separate config -> clients -> services -> tests. Wrap external calls in a client class so tests can mock at that boundary. No factories-of-factories, custom DSLs, or base classes "just in case."
- **Error handling**: `try/except` around I/O and network calls; catch specific exceptions, never bare `except:`. Raise clear domain exceptions with context. Add timeouts to every network call. Fail fast on bad config.
- **Readable over clever**: small functions (one job each), early returns over deep nesting, type hints on public functions, dataclasses instead of loose dicts. If a line needs a comment to be understood, simplify it.
- **Security**: no hardcoded secrets/URLs — read from `config.yaml` or env via the loader. Never log credentials or PII. No `shell=True`.
- **Mocking**: Use `pytest-mock` (`mocker` fixture) for all mocking. Never use `responses` or other third-party mock libraries. Patch at the HTTP client boundary (`requests.request` / your HTTP lib's request function) with a helper like `_resp()` that builds mock Response objects. This keeps mocks explicit, debuggable, and avoids extra dependencies.
- **Pytest discipline**: Arrange-Act-Assert; fixtures in `conftest.py`; `@pytest.mark.parametrize` for data cases; markers `unit`/`integration`; tests must be independent and order-free. Mock HTTP/external systems, not internal logic. Cover happy path + validation failure + one edge/error case per critical flow.
- **Comments**: explain *why*, not *what*. No narration comments.

## Step 3 — Build order

1. `config_loader.py` + `config.yaml` (with a `test` env profile)
2. One client (the thin adapter for the system under test)
3. Models for the data exchanged
4. `conftest.py` fixtures (load config via `load_config(env="test")`) + any `tests/data/` files
5. Tests (unit first, then integration)
6. `README.md` with how to install, configure, and run (`pytest`, `pytest -m unit`)

## Step 4 — Working rules

- **Start with Step 0**: Ask the user for their assignment before doing anything else.
- Generate **one file at a time**, smallest correct version. Pause for user review before the next file.
- After meaningful changes, show the exact `pytest` command to run and expected outcome.
- If unsure about a requirement, ask **one** focused question instead of guessing.
- Keep total dependencies minimal and justify any third-party library.
- Prefer finishing a small, fully-working slice over a large half-working one.

## Definition of done

- [ ] Structure matches the layout above
- [ ] `pytest` passes with documented commands
- [ ] No secrets/URLs hardcoded; config-driven
- [ ] Clear names, type hints, specific exception handling
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note (tradeoffs + what I'd add in production)

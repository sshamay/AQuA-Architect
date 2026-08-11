# python-test-starter

Scaffolds Python + pytest automation projects following Principal Automation Engineer standards.

## What it does

When loaded, the skill drives the scaffolding of a complete test automation project in a defined order:

1. **Gather requirements** — asks about the assignment (SUT type, base URL, tech stack, constraints) before writing any code
2. **Create the folder structure** — src → tests, with no business logic in tests and no test logic in src
3. **Generate one file at a time** — smallest correct version, pausing for review between files
4. **Enforce coding standards** — naming, error handling, type hints, security, mocking discipline
5. **Finish with a definition-of-done checklist** — verified pytest run, no hardcoded secrets, runnable README

## Environment model

Environments are selected by the `APP_ENV` env var — no `config.yaml`, no `.env`. A `TEST_CONFIG` registry in `tests/conftest.py` maps each env (`dev`/`qa`/`staging`/`e2e`/`prod`/`local`/`ci`) to its API URL and server strategy (`testclient`/`uvicorn`/`container`/`local`). Safety rules are enforced at import time:

- `APP_ENV` defaults to `dev`; unknown values fail fast (`pytest.exit`)
- `APP_ENV=prod` requires `--run-prod` or pytest aborts
- Secret env vars (`API_KEY`, `AUTH_TOKEN`, `DB_PASSWORD`) are stripped from the environment on import

## Folder structure it produces

```
project_root/
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── clients/           # thin API/UI adapters (HTTP calls live here)
│       ├── services/          # workflows that orchestrate clients
│       └── models/            # dataclasses for request/response data
├── tests/
│   ├── conftest.py            # APP_ENV registry, server strategies, secret stripping, prod guard
│   ├── data/                  # test-owned data: sample payloads
│   ├── unit/
│   │   └── conftest.py        # auto-marks tests in this dir as unit
│   ├── integration/
│   │   └── conftest.py        # auto-marks tests in this dir as integration
│   └── e2e/
│       └── conftest.py        # auto-marks tests in this dir as e2e
├── pyproject.toml             # pytest config (testpaths, markers) + test deps
├── requirements.txt           # pinned versions
├── README.md                  # setup, run commands, design notes
└── .gitignore
```

## Key rules it enforces

- **APP_ENV model**: environments are chosen by the `APP_ENV` env var; the `TEST_CONFIG` registry in `conftest.py` is the single source of truth for API URL + server strategy. No YAML profiles, no `.env`.
- **Server strategies**: `testclient` (in-process ASGI), `uvicorn` (spawned subprocess, xdist port offsets), `container` (docker compose), `local` (already-running server). Use temp/sandboxed resources and clean up in `finally`.
- **Config vs test data**: runtime settings live only in the `TEST_CONFIG` registry; test-owned artifacts (sample payloads) live in `tests/data/`. No duplicated config that drifts.
- **Mocking**: uses `pytest-mock` (`mocker` fixture). Patches at the HTTP client boundary (`requests.request`), never a third-party mock library like `responses`.
- **Auto-marking**: each test dir auto-marks itself via `pytest_collection_modifyitems` — no manual markers.
- **Build order**: conftest → client → models → nested conftests + test data → tests → README.
- **Coding standards**: verb-first functions, type hints, dataclasses over dicts, specific exception handling, timeouts on every network call, no hardcoded URLs/secrets, no `shell=True`.
- **Pytest discipline**: Arrange-Act-Assert, fixtures in `conftest.py`, `@pytest.mark.parametrize` for data cases, `unit`/`integration`/`e2e` markers, order-free tests, happy path + validation failure + one edge case per flow.

## Usage

### As a skill (guided building)

```text
skill("python-test-starter")
```

The assistant loads the rules and starts by asking you about the assignment, then builds the project file-by-file with your review at each step.

### As an agent (autonomous scaffolding)

```text
@python-test-starter
```

A dedicated subagent scaffolds the project from scratch, asking for the assignment first, then generating the full structure.

### Trigger phrases

The skill auto-triggers on requests like:

- "start a new Python test project"
- "scaffold a pytest project"
- "create an automation framework"

## Definition of done (checked by the skill)

- [ ] Structure matches the layout
- [ ] `pytest` passes with documented commands
- [ ] No secrets/URLs hardcoded; `APP_ENV`-driven with prod guard and secret stripping
- [ ] Clear names, type hints, specific exception handling
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note included

## Installation (opencode)

```bash
mkdir -p ~/.config/opencode/skills/
cp -r skills/python-test-starter ~/.config/opencode/skills/
```

Then quit and restart opencode.

## GitHub Copilot

### Option A — Install as a Custom Agent (recommended)

GitHub Copilot supports custom agents defined as `.agent.md` files. Install at the user level to make python-test-starter available in all your projects, or at the repository level to share with your team.

**User level** (available everywhere on your machine):

```bash
mkdir -p ~/.copilot/agents
cp skills/python-test-starter/python-starter.agent.md ~/.copilot/agents/python-starter.agent.md
```

**Repository level** (shared with your team via source control):

```bash
mkdir -p .github/agents
cp skills/python-test-starter/python-starter.agent.md .github/agents/python-starter.agent.md
```

### Option B — Install as a Skill

Copilot auto-discovers skills from `.github/skills/<name>/SKILL.md` (project) or `~/.copilot/skills/<name>/SKILL.md` (personal).

**Personal**:

```bash
mkdir -p ~/.copilot/skills
cp -r skills/python-test-starter ~/.copilot/skills/
```

**Project** (team-shared):

```bash
mkdir -p .github/skills
cp -r skills/python-test-starter .github/skills/
```

### Usage

Invoke the agent by name in Copilot Chat or Copilot CLI:

> @python-starter Scaffold a pytest project for testing our REST API.

Or let it trigger naturally:

> Start a new Python test project.

The agent asks about the assignment (SUT, base URL, tech stack, constraints) before generating any code, then scaffolds the project file-by-file with your review at each step.

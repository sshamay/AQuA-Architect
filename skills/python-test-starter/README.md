# python-test-starter

Scaffolds Python + pytest automation projects following Principal Automation Engineer standards.

## What it does

When loaded, the skill drives the scaffolding of a complete test automation project in a defined order:

1. **Gather requirements** — asks the assignment questions first (SUT type, tech stack, reliability scope, fault-injection style) before writing any code
2. **Environment model (APP_ENV)** — environments are selected by the `APP_ENV` env var from a `TEST_CONFIG` registry (topology only); credentials live in a gitignored `.env` and never in the registry
3. **Generate one file at a time** — smallest correct version, pausing for review between files
4. **Enforce coding standards** — naming, error handling, type hints, security, mocking discipline
5. **Deterministic reliability coverage** — injectable clock + scripted faults via `pytest-mock`; the SUT carries no chaos logic
6. **Finish with a definition-of-done checklist** — verified pytest run, no hardcoded secrets, runnable README

## Folder structure it produces

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

## Key rules it enforces

- **Registries vs secrets**: `TEST_CONFIG` holds only non-secret topology (API URL, server strategy, ports, node ids); every credential lives in a gitignored `.env`/CI env, loaded via `python-dotenv` and stripped from the environment after capture.
- **Safety**: `APP_ENV` defaults to `dev`; unknown values → `pytest.exit`; `APP_ENV=prod` requires `--run-prod`.
- **Mocking**: `pytest-mock` (`mocker` fixture) only. Mock at the SUT's true boundary, fake real behavior for stateful collaborators (`mocker.patch.object(node, "write_chunk", ...)`, never the whole class). Faults are scripted (`side_effect` fall-through + `FakeClock`), never `random`.
- **Build order**: pyproject/conftest registry → client → models → nested conftests + tests → `tests/faults.py` (if reliability scope) → README.
- **Coding standards**: verb-first functions, type hints, dataclasses over dicts, specific exception handling, timeouts + injectable clock, no hardcoded URLs/secrets, no `shell=True`.
- **Pytest discipline**: Arrange-Act-Assert, auto-marker nested conftests, `@pytest.mark.parametrize`, soft assertions via `pytest-check` for bulk field validation, order-free tests.

## Usage

### As a skill (guided building)

```text
skill("python-test-starter")
```

The assistant loads the rules and starts by asking the assignment questions, then builds the project file-by-file with your review at each step.

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

- [ ] Structure matches the layout (`pyproject.toml`, not `pytest.ini`)
- [ ] `pytest` passes with documented commands; env selected via `APP_ENV`, defaults to `dev`
- [ ] No secrets/URLs hardcoded; `TEST_CONFIG` holds topology only, `.env.example` committed, prod guard + secret stripping present
- [ ] Clear names, type hints, specific exception handling
- [ ] Complex-response tests validate all fields via `pytest-check` soft assertions
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note included
- [ ] If reliability scope: deterministic fault-injection + retry-with-backoff (injectable clock) + state-verification tests, mapped in the coverage matrix

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

The agent asks about the assignment (SUT, tech stack, reliability scope, constraints) before generating any code, then scaffolds the project file-by-file with your review at each step.
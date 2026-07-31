# python-test-starter

Scaffolds Python + pytest automation projects following Principal Automation Engineer standards.

## What it does

When loaded, the skill drives the scaffolding of a complete test automation project in a defined order:

1. **Gather requirements** — asks about the assignment (SUT type, base URL, tech stack, constraints) before writing any code
2. **Create the folder structure** — config → src → tests, with no business logic in tests and no test logic in src
3. **Generate one file at a time** — smallest correct version, pausing for review between files
4. **Enforce coding standards** — naming, error handling, type hints, security, mocking discipline
5. **Finish with a definition-of-done checklist** — verified pytest run, no hardcoded secrets, runnable README

## Folder structure it produces

```
project_root/
├── config/
│   ├── config.yaml            # ONE source of truth: env profiles (dev/test)
│   └── config.example.yaml    # committed template
├── src/
│   └── <package>/
│       ├── __init__.py
│       ├── config_loader.py   # load + validate YAML into typed settings
│       ├── logging_setup.py   # one place to configure logging
│       ├── clients/           # thin API/UI adapters (HTTP calls live here)
│       ├── services/          # workflows that orchestrate clients
│       └── models/            # dataclasses for request/response data
├── tests/
│   ├── conftest.py            # shared fixtures: loads config with env="test"
│   ├── data/                  # test-owned data: sample payloads
│   ├── unit/                  # fast, isolated tests with mocks
│   └── integration/           # tests hitting real/stubbed dependencies
├── pytest.ini                 # markers, testpaths
├── requirements.txt           # pinned versions
├── README.md                  # setup, run commands, design notes
└── .gitignore
```

## Key rules it enforces

- **Config vs test data**: runtime settings live only in `config.yaml` (with a `test` env profile); test-owned artifacts (sample payloads) live in `tests/data/`. No duplicated config that drifts.
- **Mocking**: uses `pytest-mock` (`mocker` fixture). Patches at the HTTP client boundary (`requests.request`), never a third-party mock library like `responses`.
- **Build order**: config_loader → client → models → conftest + test data → tests → README.
- **Coding standards**: verb-first functions, type hints, dataclasses over dicts, specific exception handling, timeouts on every network call, no hardcoded URLs/secrets, no `shell=True`.
- **Pytest discipline**: Arrange-Act-Assert, fixtures in `conftest.py`, `@pytest.mark.parametrize` for data cases, `unit`/`integration` markers, order-free tests, happy path + validation failure + one edge case per flow.

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
- [ ] No secrets/URLs hardcoded; config-driven
- [ ] Clear names, type hints, specific exception handling
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note included

## Installation

```bash
mkdir -p ~/.config/opencode/skills/
cp -r skills/aqua-architect/python-test-starter ~/.config/opencode/skills/
```

Then quit and restart opencode.

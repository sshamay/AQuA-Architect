---
name: ts-playwright-starter
description: |
  Scaffolds a TypeScript + Playwright automation project following principal-automation-engineer standards.
  Load this when the user says "start a new Playwright test project", "scaffold a Playwright project",
  "create a TypeScript automation framework", or any request to build a new TS/e2e testing codebase from scratch.
  Use ONLY for greenfield TypeScript/Playwright test projects, not for adding tests to an existing project.
---

# TypeScript + Playwright Test Starter

## Step 0 — Gather requirements first

Before scaffolding, ask the user these questions:

1. **Assignment** — What system are you testing? (UI / REST API / both) Multi-layer coverage is expected: UI, API, and integration.
2. **System under test** — What's the base URL / app path / package under test?
3. **Tech stack** — Allowed tools? (`@playwright/test` / playwright + vitest / fetch / etc.)
4. **Constraints** — Time limit, must run offline, no network, browsers available, etc.
5. **Package name** — Name for the project (default: derived from project dir)

Adapt the generated code to these answers. Do not hardcode assumptions.

## Step 1 — Folder structure

```
project_root/
├── config/
│   ├── config.yaml                        # ONE source of truth: env profiles (dev/staging/test)
│   └── config.example.yaml                # committed template
├── src/
│   ├── config/
│   │   └── loader.ts                      # load + validate config into typed settings
│   ├── logger.ts                          # one place to configure logging
│   ├── clients/                           # thin API adapters (request context / fetch)
│   ├── pages/                             # Page Object Models, one per screen
│   ├── fixtures/                          # custom Playwright fixtures (auth state, apiContext)
│   ├── models/                            # TS types/interfaces for request/response data
│   └── utils/                             # shared helpers (date/format/transform); no "just in case" code
├── tests/
│   ├── data/                              # test-owned data: sample payloads, expected JSON
│   ├── api/                               # API-layer tests using clients (requestContext/fetch)
│   ├── e2e/                               # UI-layer tests using Page Objects
│   └── integration/                       # cross-layer tests: UI flows + real API state via apiContext
├── playwright.config.ts                   # testDir, projects, reporters, retries, baseURL
├── .env.example                           # committed template (baseURL, creds, etc.)
├── package.json                           # pinned deps, scripts
├── tsconfig.json                          # strict TS
├── README.md                              # setup, run commands, design notes
└── .gitignore                             # secrets, node_modules, test-results, .env
```

**Key rule**: Keep test logic out of `src/` and business logic out of `tests/`.

**Config vs test data**: There is no `tests/config/` that re-declares runtime settings — that causes drift. Runtime settings (baseURL, credentials) come from `.env` / config profiles loaded via the shared `loader.ts`, and `playwright.config.ts` loads the same `.env`. Test-owned artifacts (sample payloads, expected responses) live in `tests/data/`, never in the app config.

## Step 2 — Coding guidelines

- **Ask before assuming**: Always begin by asking what the user is testing. Gather SUT, tech stack, and constraints before writing any code.

- **Clear naming**: camelCase for files/functions/variables, PascalCase for classes. Verb-first functions (`createOrder`, `loadConfig`). No cryptic abbreviations, no 8-word names. Test names describe behavior (`logs in and lands on dashboard`).

- **Right abstraction**: separate config -> clients/pages -> tests. Page Objects wrap UI interaction; API clients wrap HTTP calls so tests can mock at that boundary. No factories-of-factories, custom DSLs, or base classes "just in case."

- **Clean OOP**: small classes with a single responsibility; prefer composition over inheritance; no god objects. Each client/Page Object should do one job and be independently testable.

- **SOLID + OOP principles**: Encapsulation — Page Objects hide selectors and UI logic from tests. Inheritance vs composition — a base `Page` class only when several pages genuinely share behavior; otherwise use composition/utility components. Abstraction — generic reusable action helpers (typed, not stringly), but never sleep-based: always built on web-first `expect`. SOLID — Single Responsibility (one class per page/feature) and Open/Closed (add a new page/client instead of editing core framework code).

- **API pattern (automate an API call)**: use the Playwright `request` fixture / `requestContext` for HTTP. Either call it directly (`request.get(baseURL + '/orders')`, assert status + body shape) or wrap it in a typed client in `src/clients/`. Every call gets a timeout; assert on status code, JSON schema/shape, and business fields, not just "2xx".

- **UI pattern (interact with an element)**: locate via a Page Object method, prefer `getByRole`/`getByTestId`, act, then assert with web-first `expect` auto-retry. Never assert on sleep-based waits.

- **Selectors**: prefer `getByRole` / `getByTestId`, never brittle CSS strings buried in tests. Expose selectors via Page Objects, not inline in tests.

- **Deterministic UI**: use web-first `expect` assertions (auto-retry). Never `waitForTimeout`/arbitrary sleeps. Isolated state per test: fresh context/page, no order dependence.

- **Error handling**: `try/catch` around I/O and network calls; catch specific errors, never bare `catch`. Raise clear domain errors with context. Add timeouts to every network call. Fail fast on bad config. Defensive: validate inputs; never silently swallow a failure.

- **Debuggable failures**: every failure must be actionable — log context (request, params, response, page state) so a red test tells you what broke and where, not just that it broke. Prefer a clear assertion message over re-running to debug.

- **Security**: no hardcoded secrets/URLs — read from `.env` or config via the loader. Never log credentials or PII.

- **Mocking**: Use `page.route` / `context.route` to intercept API calls at the HTTP boundary. Mock external systems, not internal logic.

- **TS discipline**: strict mode (`noImplicitAny`, etc.). Type public signatures. Interfaces/types for exchanged data instead of loose objects.

- **Comments**: explain *why*, not *what*. No narration comments.

## Step 3 — Build order

1. `package.json` + `tsconfig.json` + `playwright.config.ts` (with `.env` handling)
2. Config loader (`src/config/loader.ts`) + `.env.example`
3. `src/logger.ts`
4. One Page Object (UI) or API client for the system under test
5. `src/fixtures/` custom fixtures + tests (`tests/api` first, then `tests/e2e`, then `tests/integration`)
6. `README.md` with how to install, configure, and run (`npx playwright test`, `npx playwright test --project=chromium`)

## Step 4 — Working rules

- **Start with Step 0**: Ask the user for their assignment before doing anything else.
- Generate **one file at a time**, smallest correct version. Pause for user review before the next file.
- After meaningful changes, show the exact `npx playwright test` command to run and expected outcome.
- If unsure about a requirement, ask **one** focused question instead of guessing.
- Keep total dependencies minimal and justify any third-party library.
- Prefer finishing a small, fully-working slice over a large half-working one.

## Definition of done

- [ ] Structure matches the layout above
- [ ] `npx playwright test` passes with documented commands
- [ ] No secrets/URLs hardcoded; env/config-driven
- [ ] Strict TS, clear names, web-first assertions
- [ ] README lets a reviewer run it in under 5 minutes
- [ ] Brief design-decisions note (tradeoffs + what I'd add in production)

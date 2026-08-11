---
name: sovereign-dockerfile
description: |
  Writes hardened, production-grade Dockerfiles for test environments following the "Sovereign AI" principles:
  minimal attack surface, non-root execution, layer optimization, environment parity (local == CI), and
  AI-native observability. Load this when the user says "add a Dockerfile", "containerize the tests",
  "make tests run in docker locally and in CI", "write a Dockerfile for the test suite", or any request to
  Dockerize a test/automation project or application. Use for greenfield Dockerfiles, not for debugging
  existing Docker builds.
---

# Sovereign AI Dockerfile Skill

Aligns with the mission of national cyber-defense and AI-native resilience: test environments must be
secure by default, reproducible anywhere (laptop, GitHub Actions, AWS), and observable.

## Principle 1 — Minimal Attack Surface (Security-First)

- **Base image**: always a minimal, hardened Linux base — `python:3.11-slim-bookworm`, `python:3.12-slim`,
  `node:20-alpine`, etc. Match the runtime's major version and pin the tag. Never `latest`.
- **Non-root execution**: the image must never run as root. Create a dedicated user (e.g. `dream-tester`),
  own the working directory and any mounted volume paths with that user, and `USER` before `ENTRYPOINT`.
- **Scope the copy**: copy only what the runtime needs. No `.git`, `.venv`, caches, or local secrets into the image.

## Principle 2 — Layer Optimization (Fast Delivery)

- **Dependency caching**: copy manifest files first (`pyproject.toml`, `requirements.txt`, `package.json`,
  `package-lock.json`, `pnpm-lock.yaml`), run the install step, THEN copy the source. This keeps the heavy
  install layer cached and only rebuilds on manifest changes.
- **Multi-stage builds**: separate build-time dependencies (compilers, dev libraries, browser installers)
  from the runtime environment. Final stage is a slim image with only what the entrypoint needs. Use
  `--from=<stage>` to copy artifacts/browsers into the final stage.

## Principle 3 — Environment Parity (Infrastructure-as-Code)

- **CI/CD alignment**: the Dockerfile is the single source of truth for the environment. The exact image
  runs on a laptop, in GitHub Actions, or on AWS — one build, one behavior.
- **Dynamic configuration**: use `ENV` and `ARG` for configuration that varies (target environments,
  `APP_ENV`, API keys, credentials). `ARG` for build-time, `ENV` for runtime. Never bake secrets into the
  image; pass them at run time via `--env-file`, `-e`, or CI secrets.

## Principle 4 — AI-Native Resilience & Observability

- **System dependencies**: explicitly install Linux utilities modern frameworks need (Playwright/Selenium:
  `libnss3`, `libatk`, `libatk-bridge2.0`, `libcups`, `libdrm`, `libxkbcommon`, `libxcomposite`, `libxdamage`,
  `libxfixes`, `libxrandr`, `libgbm`, `libasound2`). For Playwright prefer `playwright install --with-deps`
  inside the build stage.
- **Insight-driven logs**: the entrypoint must write test results and logs to a mounted volume
  (e.g. `/app/test-results`) so failures are investigable. Document the `-v` mount in the README.

## Step 0 — Gather requirements first

Ask the user:

1. **Stack** — Python (pytest/pytest-playwright/httpx), Node (Playwright/pnpm), etc.? Which major version?
2. **What runs** — test suite, app, or both? What is the entrypoint (e.g. `pytest`, `node dist/...`)?
3. **Runtime needs** — browsers (chromium/firefox/webkit), system libs, GPU-less (headless), network access?
4. **Config surface** — which env vars / build args vary per environment (`APP_ENV`, API keys, URLs)?
5. **Volume/logs** — where should results and artifacts land (host mount path)?

Adapt the generated Dockerfile to these answers. Do not hardcode assumptions.

## Step 1 — Reference layout

```
project_root/
├── Dockerfile            # multi-stage, non-root, cached deps
├── .dockerignore         # .git, .venv, caches, test-results, local .env
├── docker-entrypoint.sh  # thin entrypoint: runs tests, writes to mounted volume
└── README.md             # build + run commands (local and CI identical)
```

## Step 2 — Golden pattern (Python/pytest + Playwright)

```dockerfile
# ── Stage 1: build — install Python deps + browsers into a venv (heavy, cached on manifest change) ──
FROM python:3.11-slim-bookworm AS build

WORKDIR /app

ENV VIRTUAL_ENV=/opt/venv \
    PLAYWRIGHT_BROWSERS_PATH=/ms-playwright

RUN python -m venv "$VIRTUAL_ENV"
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

COPY pyproject.toml ./
RUN pip install --no-cache-dir -e ".[test]" \
    && python -m playwright install --with-deps chromium

# ── Stage 2: minimal runtime, non-root ──
FROM python:3.11-slim-bookworm AS runtime

WORKDIR /app

ENV APP_ENV=dev \
    PYTHONUNBUFFERED=1 \
    VIRTUAL_ENV=/opt/venv \
    PLAYWRIGHT_BROWSERS_PATH=/ms-playwright \
    PATH="/opt/venv/bin:$PATH"

COPY --from=build /opt/venv /opt/venv
COPY --from=build /ms-playwright /ms-playwright
COPY . .

RUN python -m playwright install-deps chromium \
    && useradd --create-home --shell /usr/sbin/nologin dream-tester \
    && mkdir -p /app/test-results \
    && chmod +x /app/docker-entrypoint.sh \
    && chown -R dream-tester:dream-tester /app

USER dream-tester

ENTRYPOINT ["/app/docker-entrypoint.sh"]
```

```sh
#!/bin/sh
set -e
exec python -m pytest -p no:cacheprovider --junitxml=/app/test-results/junit.xml "$@"
```

**Invariants:**

- Install everything into a venv (`/opt/venv`) in the `build` stage and `COPY --from=build /opt/venv /opt/venv` — a plain `COPY --from=build /app /app` does NOT carry installed packages. Set `PATH` so the entrypoint finds the venv.
- Browsers go to `PLAYWRIGHT_BROWSERS_PATH` (set in BOTH stages), so `COPY --from=build /ms-playwright /ms-playwright` works. `playwright install --with-deps` runs in build (downloads + apt libs); the runtime stage re-runs `playwright install-deps chromium` (apt libs only, no download) because `--with-deps` apt packages do NOT survive the stage boundary.
- The install + browser-deps layer sits in `build`, so source edits rebuild in seconds; manifest edits rebuild deps only.
- Final stage is the same base tag — thin, no compilers, no root.
- Entrypoint writes results into the mounted volume; `--junitxml` gives CI consumable output.
- `docker-entrypoint.sh` must be executable in the image (`RUN chmod +x docker-entrypoint.sh`).

## Step 3 — Build & run (identical locally and in CI)

```bash
docker build -t dream-tests:latest .

# local: run against the live env, results land in ./test-results
docker run --rm \
  -e APP_ENV=dev \
  -v "$(pwd)/test-results:/app/test-results" \
  dream-tests:latest -q

# CI (GitHub Actions) — same command, secrets via env, not baked in
docker run --rm \
  -e APP_ENV=ci \
  -e DREAM_EMAIL="${{ secrets.DREAM_EMAIL }}" \
  -v "$GITHUB_WORKSPACE/test-results:/app/test-results" \
  dream-tests:latest -m e2e --headed=false
```

- Prefer `docker run --rm`; map a host directory over `/app/test-results`.
- For headed/`--slowmo` debugging on a laptop: pass extra args after the image name (they reach the entrypoint's `"$@"`).

## Step 4 — Working rules

- Generate **one file at a time** (Dockerfile → .dockerignore → entrypoint), smallest correct version. Pause for review.
- Show the exact `docker build` / `docker run` commands and expected outcome after meaningful changes.
- Never hardcode secrets into the Dockerfile; document env vars instead.
- If unsure about a requirement, ask **one** focused question instead of guessing.
- Keep dependencies minimal and justify every `apt-get`/`pip` package.

## Definition of done

- [ ] Non-root user runs the entrypoint (no `USER root` anywhere after build)
- [ ] Manifest copied before source (install layer cached)
- [ ] Multi-stage build; final image slim, same base family as build stage
- [ ] Env vars/`ARG`s drive configuration; no secrets baked in
- [ ] Browsers/system libs installed via `--with-deps` (or explicit lib list)
- [ ] Entrypoint writes results/logs to a mounted volume
- [ ] README documents build + run so `docker build`/`docker run` work locally and in CI identically

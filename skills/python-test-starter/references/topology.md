# Topology module — multi-node & chaos proxy (phase 2)

**Read this only if Step 0 Q7 selected the real multi-node option.** Phase 1 (deterministic fault injection, `reliability-patterns.md`) should exist first — it covers most failure modes with zero infrastructure. Graduate to this module when a fake cannot express the failure: half-open TCP, `kill -9` mid-ack, OS-level packet loss between real processes.

## When to graduate

- Deterministic fakes can't express a real failure (half-open TCP connections, `kill -9` mid-acknowledgement, OS-level packet loss between real processes).
- You need to test the actual `container` server strategy against node death, not a mocked client.

## Docker Compose topology

Extend the existing `container` server strategy from `tests/conftest.py`:

```yaml
# docker-compose.chaos.yml
services:
  app-1: { build: ., environment: [NODE_ID=1] }
  app-2: { build: ., environment: [NODE_ID=2] }
  chaos-proxy:            # sits between client and app nodes
    image: alpine/socat   # or a small custom TCP proxy
    command: TCP-LISTEN:8080,fork TCP:app-1:8000
```

The chaos proxy injects latency/drops at the TCP level, configured per test via env or a control endpoint. This is *above* the HTTP-client boundary — it catches failures the scripted scheduler cannot fake.

## Node kill / restart

- **Kill:** `docker compose stop app-1` mid-test → assert the client fails over/retries to `app-2`.
- **Restart:** `docker compose start app-1` → assert state converges (read-your-writes holds).
- **Teardown:** always `docker compose down -v` in `finally` / fixture teardown — matches the core skill's temp-resource rule.

## Readiness

Reuse the core skill's bounded polling pattern (`while time.monotonic() < deadline`) per node — wait for readiness before each phase. A node that never comes back is a test failure with context, not a hang.

## Cost warning

This phase is slower, flakier, and Docker-dependent. Gate everything behind a marker:

```python
@pytest.mark.chaos
def test_survives_node_kill(...): ...
```

Default `pytest` stays fast (unit + deterministic reliability); run `pytest -m chaos` explicitly — CI nightly, not per-commit. Record every chaos row in the strategy matrix with its "infrastructure" cost tag.

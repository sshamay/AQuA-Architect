# Reliability patterns — deterministic fault injection, retries, state

Loaded when Step 0 Q6 selected any reliability option. All patterns here are **deterministic and infrastructure-free**: fake clock + scripted faults, injected with `pytest-mock` at the SUT's true boundary — the HTTP request function (section 2) or, for in-process/library SUTs, the component's method call (section 3). No real sleeps, no `random`, no Docker in this module — multi-node chaos lives in `topology.md` (phase 2).

## 1. Injectable clock (fake time)

Never assert on wall-clock backoff. Inject time into the client under test:

```python
class FakeClock:
    def __init__(self) -> None:
        self.now = 0.0
        self.sleeps: list[float] = []

    def sleep(self, seconds: float) -> None:
        self.sleeps.append(seconds)
        self.now += seconds
```

The client takes `clock: Clock` as a constructor dependency (default: real `time`). In tests, pass `FakeClock` — a 2-second backoff test then runs in ~0 ms and asserts the **exact schedule**: `clock.sleeps == [1.0, 2.0, 4.0]`.

## 2. FaultScheduler — scripted faults at the HTTP boundary

Patch the same boundary the core skill mocks (`requests.request` / your HTTP lib's request function), but route through a scheduler that replays a **scripted sequence** instead of a static mock:

```python
class FaultScheduler:
    """Replays scripted outcomes in order; falls through to success when empty."""

    def __init__(self, script: list[BaseException | Response]) -> None:
        self.script = list(script)

    def __call__(self, *args, **kwargs):          # stands in for requests.request
        if self.script:
            item = self.script.pop(0)
            if isinstance(item, BaseException):
                raise item
            return item
        return _resp(200, {...})                  # the core skill's _resp() helper
```

Script vocabulary (maps to the strategy matrix distributed column):

| Failure mode | Script entry |
|---|---|
| Latency spike | entry that advances `FakeClock` past the client's timeout, then returns/raises `Timeout` |
| Packet loss / disconnect | `ConnectionError` / `ConnectError` |
| Timeout | `Timeout` / `ReadTimeout` |
| Transient 5xx | `_resp(503, ...)` |
| Race between two writers | two tests (or threads) sharing one scheduler, ordering controlled by the script — assert final state, not timing |

Determinism rule: same script → same outcome, every run. If a test needs `sleep` to "let things settle," it needs a fake clock or a barrier instead.

## 3. In-process SUTs (library / distributed component — no HTTP client)

When the SUT is an in-repo library or a distributed component (the Q6 "Distributed component" assignment), the boundary to mock is a **method call**, not `requests.request`. Two rules keep it deterministic and meaningful:

- **The SUT carries no chaos logic.** No `failure_rate`, no seeded `random` inside the SUT — all fault injection lives in the tests. A `write_chunk` that "sometimes fails" makes tests slow and flaky; a plain deterministic method makes every test a fast, reproducible assertion.
- **Patch the method, never the class.** Keep the object real (so read-back, `has_chunk`, and checksum state assertions test actual SUT logic) and patch only its boundary method with `mocker.patch.object`:

```python
def scripted_writes(mocker, node, *outcomes):
    """Replay a transient fault sequence, then fall through to the real write."""
    real_write = node.write_chunk
    queue = list(outcomes)

    def play(chunk):
        if queue:
            outcome = queue.pop(0)
            if outcome is not None:
                raise outcome
        real_write(chunk)

    return mocker.patch.object(node, "write_chunk", side_effect=play)


def down_writes(mocker, node):
    """Fail permanently — a dead node that never acks and never stores."""
    return mocker.patch.object(
        node, "write_chunk",
        side_effect=StorageNodeError(f"{node.node_id}: node unreachable"),
    )
```

`scripted_writes` mirrors a node that recovers (faults replay, then the real write stores the chunk); `down_writes` mirrors a dead node the controller must give up on after `max_attempts`. Pair with the injectable `FakeClock` (section 1) passed as the controller's `sleep=` dependency so backoff tests assert the exact schedule in ~0 ms. This is the pattern in the MultiNodeDataReplicator scaffold.

## 4. Retry with exponential backoff — the test pattern

```python
def test_retry_backoff_then_success(clock, client, mocker):
    sched = FaultScheduler([_resp(503, {}), _resp(503, {}), _resp(200, {"ok": True})])
    mocker.patch("pkg.client.requests.request", side_effect=sched)

    result = client.get_with_retry("/health", clock=clock)

    assert result["ok"] is True
    assert clock.sleeps == [1.0, 2.0]          # exponential schedule, not real sleep
    assert sched.script == []                   # all faults consumed
```

Cover with tests: **eventual success** (above), **exhausted retries** (script of all-503s → typed domain exception after N attempts, schedule length N-1), **no retry on permanent errors** (`_is_retryable` returns False for 400/401/404 — fails fast, `clock.sleeps == []`).

### Transient vs permanent classifier

```python
def _is_retryable(exc: Exception, resp: Response | None = None) -> bool:
    if resp is not None:
        return resp.status_code in {429, 500, 502, 503, 504}
    return isinstance(exc, (Timeout, ConnectionError))   # transient by nature
```

Unit-test the classifier directly with `@pytest.mark.parametrize` — it is the decision point for every retry.

## 5. Idempotency & state verification

A retry test that only asserts "eventually 200" proves nothing about safety. Always pair it with a **state assertion** on a fake store/counter:

```python
def test_retry_does_not_duplicate_side_effect(clock, client, store, mocker):
    sched = FaultScheduler([_resp(503, {}), _resp(200, {"id": 7})])
    mocker.patch("pkg.client.requests.request", side_effect=sched)

    client.create_order(payload, clock=clock)

    check.equal(store.count_calls("create_order", payload), 1, msg="side effect exactly once")
    check.equal(len(clock.sleeps), 1, msg="one backoff before success")
```

State-verification patterns (strategy matrix, distributed column):

- **N-in / N-out**: enqueue N jobs → run worker → assert store holds exactly N results, zero duplicates.
- **Read-your-writes**: write via node A → read via node B (or after simulated reconnect) → value must be visible.
- **Failure-state tracking**: client records `attempts` / `last_error` — assert those fields, not just the raised exception.

Use `pytest-check` soft assertions for multi-invariant state reports (count + no-duplicates + attempt log) so one red field doesn't hide the rest.

## 6. Async variant (only if Q6 selected async)

- Same FakeClock/FaultScheduler shapes; the scheduler raises/returns inside the patched **async** request function.
- Race tests coordinate with `asyncio.Event` / `asyncio.Barrier` — never `sleep`.
- Dependencies: `pytest-asyncio` in `requirements.txt`; add `asyncio_mode = "auto"` to `[tool.pytest.ini_options]`.

## What NOT to do

- No `time.sleep(2)` to "wait for" backoff — inject the clock.
- No `random.random()` flakiness — script the fault sequence.
- No chaos logic (failure-rate / randomness) inside the SUT — faults live in the tests.
- No real multi-node infrastructure here — that is `topology.md` (phase 2, only if Q7 chose it).

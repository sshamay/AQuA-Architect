# Coverage strategy matrix

Fill this matrix **before writing tests** when reliability scope is selected in Step 0 (Q6). Every planned behavior gets one row; every failure mode that applies gets a cell. Cells map to test ids (`test_<behavior>_<expected_outcome>`).

## The four categories

| Category | What it covers | Examples |
|---|---|---|
| **Happy path** | The one obvious correct flow | `POST /orders` → 201, body matches contract |
| **Boundary** | Limits just inside/outside valid | empty payload, max length, off-by-one ids, unicode, 0 vs 1 items |
| **Error handling** | Invalid input or rejected request → clean, typed failure | 400/404/409 → domain exception, not a crash; validation messages preserved |
| **Distributed / failure modes** | The network lies, nodes die, time passes | latency spike, packet loss, disconnect mid-response, race between two writers, retry causing duplicate delivery |

## Matrix template

| Behavior | Happy | Boundary | Error | Distributed / failure |
|---|---|---|---|---|
| `create_order` | ✓ 201 + body | empty body → 400 | conflict → typed `ConflictError` | latency → timeout → retry → **state effect exactly once** |
| `poll_status` | ✓ terminal state | already-terminal → no-op | 404 → typed | disconnect → reconnect → read-your-writes holds |
| … | | | | |

## Rules

- **Distributed column is mandatory** when Q6 selected any reliability option — don't leave it blank; write "N/A: single-process, no network" only with a reason.
- One test id per cell; leave cells empty only if the mode genuinely cannot occur (document why in the design note).
- Pair each distributed cell with its **state assertion**: a retry test without an idempotency check proves nothing.
- The matrix lives in the project README (or `tests/COVERAGE.md`) — it is the reviewable artifact of your test plan.

## What cost means here (in the matrix footer)

Record for each filled row whether verification is **free** (deterministic, in-process, `pytest -m unit`) or **infrastructure** (needs spawned server / container / chaos proxy). Keep the free column dominant; infrastructure rows are the exception and get their own marker.

# Concepts to review beforehand

The scaffold this skill builds assumes fluency in four concept families. Review these before starting — each row maps to where you'll see it in the generated project.

| Concept family | Where it appears in the scaffold |
|---|---|
| **Testing frameworks** — pytest fixtures, `@pytest.mark.parametrize`, setup/teardown, custom assertions | Fixtures in `conftest.py`; `@pytest.mark.parametrize` for data/boundary cases; `pytest-check` soft assertions (`msg=` per field) for bulk validation; teardown in `finally` / fixture cleanup |
| **Concurrency & timing** — threading / multiprocessing / asyncio basics, polling loops, timeout conditions | Bounded polling loop with overall deadline for readiness checks (`while time.monotonic() < deadline`); timeouts on every network call; `pytest-asyncio` when async is in scope (reliability module) |
| **OOP & design patterns** — Builder/Factory for test data, wrapper classes for mock interfaces | Client wrappers around the HTTP boundary (the mock/patch point); dataclass models for request/response data; test-data builders live in `tests/data/` and fixtures (factories for payloads **are** encouraged — the "no factories" rule bans factories-of-factories, not test-data builders) |
| **Error handling & retries** — custom exception classes, tracking failure states, transient vs permanent errors | Domain exceptions raised with context; `_is_retryable(exc)` classifier separating transient (retryable) from permanent (fail fast) errors; attempt/failure state tracked on the client for assertions (reliability module) |

If any row feels unfamiliar, skim the referenced part of SKILL.md before scaffolding — the generated code will use these patterns directly.

# Test Design Review Checklist

Before generating code, verify coverage.

## Prevent (P1, P2)

- [ ] Prompt validation
- [ ] Tool contracts
- [ ] Agent workflow
- [ ] Runtime constraints
- [ ] Golden dataset defined

## Detect (P3, P4, P5, P6)

- [ ] JSON Schema validation
- [ ] Business rules
- [ ] Semantic similarity
- [ ] LLM Judge
- [ ] Retrieval validation
- [ ] Tool invocation verification

## Govern (P7, P8)

- [ ] Confidence scoring
- [ ] Business thresholds
- [ ] HITL routing
- [ ] Conflicting evaluator resolution

## Observe (P9)

- [ ] Retrieval telemetry
- [ ] Tool call telemetry
- [ ] Prompt version telemetry
- [ ] Token usage / latency / cost
- [ ] Trace ID propagation
- [ ] Telemetry assertions in tests

## Learn (P10)

- [ ] Production feedback collection
- [ ] Golden Dataset versioning
- [ ] Regression execution in CI/CD
- [ ] Model upgrade re-testing
- [ ] Prompt drift detection

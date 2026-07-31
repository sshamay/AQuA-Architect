# Operating Modes

## Mode Selection

| Condition | Mode |
|-----------|------|
| No architecture/design exists yet, or a testing strategy is requested before implementation | Design |
| Architecture exists and the user asks to implement or write tests | Build |
| An implemented AI system exists and the user asks to review, score, or audit it | Review |

If the condition is unclear, ask the user which mode they want.

---

## Mode 1 — Design

Goal: design quality before implementation.

Questions to ask:
- **Architecture** — which LLM? which orchestration framework? which vector database? which embedding model? which tools? is there retrieval/memory? are prompts versioned?
- **Business** — what business problem is solved? which workflows are critical? which failures are unacceptable?
- **Risk** — what level of confidence is required? what is the business risk tolerance?
- **Telemetry** — what pipeline stages will be instrumented? what telemetry must be asserted?
- **Golden Dataset** — what scenarios, edge cases, and production failures must anchor regression?

Output: architecture summary, risk register, testing strategy mapped to the five pillars, telemetry plan, golden dataset plan.

---

## Mode 2 — Build

Goal: guide implementation.

- Map the architecture first (components, interfaces, tools)
- Generate code per component
- Suggest assertions and evaluation tests
- Recommend telemetry instrumentation
- Verify coverage against the Test Design Review checklist before generating code

Output: code with assertions, test cases with expected outcomes, telemetry instrumentation recommendations.

---

## Mode 3 — Review

Goal: review an existing AI system and identify gaps.

Process:
1. Architecture Review
2. Quality Review
3. Risk Review
4. Observability Review
5. Regression Review

Produce:
- Architecture Score (per pillar)
- Coverage report
- Missing pillars
- Top risks (with severity)
- ROI improvements (ordered by impact/effort)

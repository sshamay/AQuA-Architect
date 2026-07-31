# Pillar 4 — Observe

## Concept

Traditional monitoring focuses on application inputs, outputs, and infrastructure metrics. AI applications require deeper observability: instrument the entire execution pipeline — retrieval, orchestration, tool execution, evaluation, and governance. Observe provides the "black box recorder" for AI systems, allowing engineers to reconstruct every execution step and determine precisely where a pipeline deviated.

The objective shifts from "Did the system work?" to "How did the system reach this result?"

**Principle:** P9 Deep Observability

## P9 — Deep Observability

Instrument every stage of the AI pipeline by emitting structured telemetry and distributed traces that capture the complete execution path rather than only the final outcome.

### Implementation

```python
def instrument_ai_pipeline(user_query):
    context = retrieve_context(user_query)
    emit_retrieval_trace(context)

    execution = orchestrate_agent(user_query, context)
    emit_orchestration_trace(execution)

    result = execute_tools(execution)
    emit_tool_execution_trace(result)

    response = generate_response(result)
    emit_operational_metrics(response)

    return response
```

## Telemetry Points

Every stage should emit structured telemetry using a shared **Trace ID** so a complete execution can be reconstructed end-to-end.

| Stage | Telemetry to capture |
|-------|----------------------|
| Retrieval | retrieved document IDs, similarity scores, retrieval latency, context size |
| Orchestration | workflow transitions, selected agents, routing decisions, execution path |
| Tool Execution | selected tool, parameters, execution status, latency, returned values, exceptions |
| LLM Invocation | model name, prompt version, token usage, latency, finish reason |
| Evaluation | semantic score, groundedness score, hallucination score, judge decision |
| Governance | confidence score, release decision, HITL routing decision |

## Implementation Guidance

- Use distributed tracing frameworks (e.g., OpenTelemetry) with a shared Trace ID
- Consider AI observability platforms: Langfuse, LangSmith, Phoenix, Helicone
- Instrument every stage, not just the final response

## QA Responsibilities

QA automation should validate not only the final response but also the telemetry emitted throughout execution. Typical assertions:

- Was the expected retrieval executed?
- Were the correct documents retrieved?
- Did the agent follow the expected orchestration path?
- Was the correct tool invoked with the expected parameters?
- Were evaluation scores generated?
- Did the governance layer make the expected release decision?
- Were latency, token usage, and cost recorded?
- Does every telemetry event share the same Trace ID?

Instead of verifying only what the AI answered, the test verifies **how** the application produced the answer.

## Why Deep Observability Matters

AI failures often originate long before the final response is generated. With structured telemetry and a shared trace identifier across retrieval, orchestration, tool execution, evaluation, and governance, every AI request becomes fully traceable from input to final decision. This allows engineers to:

- reconstruct the complete execution path,
- rapidly isolate root causes,
- verify expected agent behavior,
- correlate production failures with the corresponding regression scenarios in the Golden Anchor.

## Questions to Ask

- Is telemetry emitted at every pipeline stage?
- Does every telemetry event share a Trace ID?
- Can these traces be asserted during automated tests?
- If not — recommend adding instrumentation before continuing.

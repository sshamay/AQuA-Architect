# Telemetry

This document defines the telemetry strategy for AQuA-Architect systems.

---

## Telemetry Types

### 1. Logs

Structured, timestamped records of discrete events.

**Format:**
```json
{
  "timestamp": "2026-07-22T12:00:00Z",
  "level": "info",
  "message": "LLM call completed",
  "service": "ai-engine",
  "correlation_id": "abc-123",
  "trace_id": "def-456",
  "duration_ms": 1250,
  "model": "gpt-4",
  "tokens_used": 1500,
  "status": "success"
}
```

**Severity Levels:**

| Level    | When to Use                                    |
|----------|------------------------------------------------|
| `error`  | System errors requiring immediate attention     |
| `warn`   | Degraded conditions that need monitoring        |
| `info`   | Normal business events and operations           |
| `debug`  | Detailed diagnostic information                 |
| `trace`  | Granular execution flow for debugging           |

**Rules:**
- Always use structured JSON format
- Include correlation_id in every log entry
- Never log sensitive data (PII, secrets, tokens)
- Use consistent field names across all services

### 2. Metrics

Numerical measurements aggregated over time.

**Types:**

| Type      | Use Case                          | Example                          |
|-----------|-----------------------------------|----------------------------------|
| Counter   | Cumulative counts                 | Total requests processed         |
| Gauge     | Current value                     | Active connections               |
| Histogram | Distribution of values            | Request latency distribution     |
| Summary    | Precomputed quantiles            | p95 latency                      |

**Naming Convention:**
```
{namespace}_{subsystem}_{metric_name}_{unit}
```

Examples:
- `aqua_ai_engine_llm_latency_ms`
- `aqua_api_requests_total`
- `aqua_rag_retrieval_results_count`

**Labels:**
- Use consistent label names
- Limit cardinality (< 1000 unique combinations)
- Include: service, status, model, operation

### 3. Traces

End-to-end tracking of requests across service boundaries.

**Span Attributes:**

| Attribute          | Required | Description                       |
|--------------------|----------|-----------------------------------|
| `span.name`        | Yes      | Human-readable operation name     |
| `span.kind`        | Yes      | client / server / internal        |
| `trace.id`         | Yes      | Unique trace identifier           |
| `span.id`          | Yes      | Unique span identifier            |
| `parent.span.id`   | Yes*     | Parent span (if exists)           |
| `duration.us`      | Yes      | Span duration in microseconds     |
| `status.code`      | Yes      | OK / ERROR                        |
| `error.message`    | No       | Error message if status is ERROR  |

**Traced Operations:**
- HTTP request handling
- LLM calls
- Database queries
- External API calls
- File I/O
- Tool executions

### 4. Events

Discrete occurrences of business or system significance.

**Event Types:**

| Category     | Examples                                  |
|-------------|-------------------------------------------|
| Business    | User signup, Purchase, Content created     |
| System      | Service started, Config changed, Deploy    |
| AI          | Prompt sent, Response received, Eval run   |
| Security    | Auth failed, Permission denied, Anomaly    |

**Event Schema:**
```json
{
  "event_type": "ai.prompt_sent",
  "timestamp": "2026-07-22T12:00:00Z",
  "service": "ai-engine",
  "correlation_id": "abc-123",
  "properties": {
    "model": "gpt-4",
    "prompt_tokens": 500,
    "temperature": 0.7
  }
}
```

---

## Correlation

All telemetry shares a common correlation_id for end-to-end tracking.

```
Request → [Service A] → [Service B] → [Service C] → Response
  │           │              │              │
  └─── correlation_id: abc-123 ─────────────┘
```

**Propagation:**
- HTTP: `X-Correlation-ID` header
- Message queues: Message attribute
- Logs: `correlation_id` field
- Traces: `trace.id` field

---

## Implementation

### OpenTelemetry Integration

```typescript
// Initialize
const tracer = trace.getTracer('aqua-ai-engine');
const meter = metrics.getMeter('aqua-ai-engine');

// Create span
const span = tracer.startSpan('llm.call', {
  attributes: {
    'llm.model': 'gpt-4',
    'llm.tokens.input': promptTokens,
  }
});

// Record metric
histogram.record(latencyMs, {
  model: 'gpt-4',
  status: 'success'
});

// End span
span.setStatus({ code: SpanStatusCode.OK });
span.end();
```

### Log Configuration

```json
{
  "version": 1,
  "formatters": {
    "json": {
      "class": "logging.Formatter",
      "format": "%(asctime)s %(levelname)s %(message)s %(correlation_id)s"
    }
  },
  "handlers": {
    "stdout": {
      "class": "logging.StreamHandler",
      "formatter": "json",
      "stream": "ext://sys.stdout"
    }
  },
  "root": {
    "level": "INFO",
    "handlers": ["stdout"]
  }
}
```

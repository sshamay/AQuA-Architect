# Observability

Observability is the ability to understand the internal state of a system by examining its external outputs. This document defines the observability strategy for AQuA-Architect systems.

---

## Three Pillars of Observability

### 1. Logs

Discrete event records for debugging and auditing.

**Structured Logging Requirements:**
- JSON format for all logs
- Consistent field names across services
- Correlation IDs for request tracing
- Appropriate severity levels

**Key Fields:**

```json
{
  "timestamp": "ISO-8601",
  "level": "info|warn|error|debug",
  "service": "service-name",
  "correlation_id": "uuid",
  "message": "Human-readable message",
  "error": {
    "type": "ErrorType",
    "message": "Error details",
    "stack": "Stack trace"
  },
  "context": {}
}
```

### 2. Metrics

Aggregated numerical measurements for monitoring and alerting.

**Essential Metrics:**

| Category   | Metrics                                    |
|------------|--------------------------------------------|
| Traffic    | Request rate, Error rate, Saturation       |
| Latency    | p50, p95, p99, Maximum                     |
| AI-Specific| Token usage, Model latency, Quality scores |
| Business   | Task completion, User satisfaction          |

### 3. Traces

End-to-end request tracking across service boundaries.

**Tracing Requirements:**
- Automatic context propagation
- Manual span creation for key operations
- Error and exception recording
- Latency attribution

---

## AI-Specific Observability

### LLM Call Tracking

Every LLM interaction is logged with:

```json
{
  "operation": "llm.call",
  "model": "gpt-4",
  "prompt_tokens": 500,
  "completion_tokens": 200,
  "total_tokens": 700,
  "latency_ms": 1500,
  "temperature": 0.7,
  "status": "success",
  "quality_score": 0.85,
  "correlation_id": "abc-123"
}
```

### Prompt Version Tracking

Every prompt template is versioned and tracked:

```json
{
  "prompt_id": "rag-generator-v2",
  "version": "2.1.0",
  "hash": "sha256:abc...",
  "model": "gpt-4",
  "parameters": {
    "temperature": 0.7,
    "max_tokens": 1000
  }
}
```

### Quality Metrics

Track AI output quality continuously:

| Metric                  | Method                      |
|-------------------------|-----------------------------|
| Faithfulness            | Citation verification       |
| Relevance               | Semantic similarity         |
| Safety                  | Classifier scoring          |
| Hallucination rate      | Ground truth comparison     |

---

## Alerting

### Alert Levels

| Level      | Response Time | Escalation          |
|------------|---------------|---------------------|
| Critical   | Immediate     | On-call engineer     |
| High       | 15 minutes    | Team lead            |
| Medium     | 1 hour        | Team review          |
| Low        | Next business day | Backlog          |

### Alert Fatigue Prevention

- Group related alerts
- Use severity-based routing
- Implement alert deduplication
- Regular alert review and cleanup

---

## Dashboards

### Service Dashboard

Essential views for every service:

1. **Overview** — Request rate, error rate, latency
2. **AI Operations** — Model usage, token consumption, quality
3. **Errors** — Error distribution, top errors, trends
4. **Dependencies** — External service health, latency
5. **Business** — Key metrics, conversion, satisfaction

### Dashboard Requirements

- Real-time data (5-second refresh)
- Historical context (24h, 7d, 30d views)
- Drill-down capability
- Mobile-friendly layout

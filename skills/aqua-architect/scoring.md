# AQuA Coverage Report

Always finish with an AQuA Coverage Report scored against the five pillars.

## Format

```
AQuA Coverage Report
====================

Prevent    ★★★★★
Detect     ★★★★☆
Govern     ★★★★☆
Observe    ★★★☆☆
Learn      ★★☆☆☆

Overall Coverage: 82%

Missing:
- Tool contract validation
- Retrieval assertions
- Telemetry validation
- Confidence thresholds
- Golden Dataset regression
```

## Star Rating

| Rating | Meaning |
|--------|---------|
| ★ | Not implemented / absent |
| ★★ | Present but incomplete |
| ★★★ | Adequate, production-viable |
| ★★★★ | Strong |
| ★★★★★ | Excellent |

## Pillar Scoring Dimensions

| Pillar | Assessed On |
|--------|-------------|
| Prevent | Prompt validation, tool contracts, agent workflow, runtime constraints, golden dataset |
| Detect | Deterministic, semantic, LLM-judge layers; progressive evaluation |
| Govern | Confidence scoring, business thresholds, HITL routing |
| Observe | Telemetry coverage per stage, trace ID propagation, telemetry assertions |
| Learn | Feedback loop, golden dataset evolution, CI regression |

## Overall Coverage Calculation

`Overall Coverage = (sum of star ratings / 25) × 100`

## Missing Items

Always list concrete missing items grouped by pillar. Every missing item should map to a recommendation that strengthens one or more AQuA pillars.

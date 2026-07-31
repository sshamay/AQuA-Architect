# Pillar 1 — Prevent

## Concept

Quality begins before a single token is generated. Prevent functions as a static analysis engine that catches configuration and architectural defects during the Build/PR stage. These deterministic gates ensure the system is structurally valid before it ever encounters a probabilistic user query.

**Principles:** P1 Shift-Left Validation, P2 Golden Datasets

## P1 — Shift-Left Validation

Catch configuration errors, broken dependencies, and architectural flaws (prompt syntax, invalid tool schemas) before execution.

### Validation Areas

**Prompt Integrity** — Block PRs when prompt templates are incomplete:
- Missing required orchestration variables (e.g., `{context}`, `{user_query}`)
- Invalid template syntax
- Missing mandatory system instructions

**Tool Contract Validation** — Verify every tool exposes a complete, standards-compliant interface contract (schema, parameters, input/output definitions) so it can be invoked reliably by the LLM and integrated safely with downstream services.

**Agent Workflow Validation** — Statically analyze routing logic for circular dependencies, unreachable nodes, invalid transitions, and non-terminating execution paths ("dead-ends").

**Operational Boundary Validation** — Estimate worst-case context consumption by combining configured prompt size, retrieval settings, conversation history, and reserved response tokens. Ensure maximum capacity stays within the selected model's operational limits.

### Implementation

```python
def shift_left_validation(prompt_template,
                          tool_definitions,
                          agent_graph,
                          max_token_limit=4096):
    validate_prompt_template(prompt_template)
    validate_tool_contracts(tool_definitions)
    validate_agent_workflow(agent_graph)
    validate_runtime_constraints(prompt_template,
                                 retrieval_config, memory_config, model_config)
    return ValidationReport.success()
```

- `validate_prompt_template()` — template engine parser (Jinja2) or custom placeholder validation
- `validate_tool_contracts()` — JSON Schema validators (jsonschema) or model validation (Pydantic)
- `validate_agent_workflow()` — graph libraries (NetworkX) or custom DFS traversal for cycle detection
- `validate_runtime_constraints()` — tokenizer libraries (tiktoken) with app config and model metadata

## P2 — Golden Datasets

Maintain a version-controlled regression suite representing business scenarios, edge cases, and known production failures. It acts as the **Golden Anchor**, stored in source control and versioned with every prompt or orchestration change.

### Golden Dataset Test Case

A single entry defines the entire "contract" for a scenario:

```json
{
  "test_id": "TC-HR-042",
  "input": "What is the paternity leave policy in the UK?",
  "risk": "High",
  "minimum_confidence": 0.95,
  "expected_behavior": {
    "tool_call": "SearchPolicyDB",
    "required_params": {"region": "UK", "topic": "paternity"},
    "output_schema": {
      "type": "object",
      "properties": {
        "weeks": {"type": "number"},
        "pay_rate": {"type": "string"}
      },
      "required": ["weeks", "pay_rate"]
    }
  },
  "ground_truth": {
    "context_id": "uk_handbook_v2_p45",
    "reference_answer": "UK employees are entitled to 2 weeks of paternity leave at £184.03 per week."
  }
}
```

Every change to prompts, orchestration, retrieval logic, or model version must execute this regression suite automatically within CI/CD before deployment.

## Questions to Ask

- Are prompt templates version controlled?
- Are mandatory variables validated?
- Can missing placeholders be detected?
- Are system prompts protected?
- Are tool schemas validated? Are required parameters enforced? Are response schemas defined?
- Can routing be statically analyzed? Can loops be detected? Are terminal nodes defined?
- Can the application fit inside the model context window? Are token budgets estimated? Is worst-case retrieval considered?

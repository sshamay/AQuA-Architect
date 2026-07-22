# Tool Calling Example

An AI agent that dynamically selects and executes external tools based on task requirements.

## Architecture

```
User Request → Intent Parser → Tool Selector → Tool Executor → Result Processor → Response
                                        ↓              ↓
                                 Tool Registry    Execution Sandbox
                                 (Schema + Auth)  (Timeout + Rollback)
```

## Components

### Intent Parser

Extracts tool-relevant intent from user requests.

- Intent classification
- Parameter extraction from natural language
- Ambiguity detection and clarification prompts

### Tool Registry

Catalogs available tools with their schemas and capabilities.

- JSON Schema for tool parameters
- Authentication requirements
- Rate limits and quotas
- Version management

### Tool Selector

Matches parsed intent to appropriate tools.

- Capability-based matching
- Multi-tool chaining for complex requests
- Fallback tool selection
- Cost-aware tool selection

### Tool Executor

Runs tools in a controlled environment.

- Sandboxed execution
- Timeout enforcement
- Resource limits
- Retry logic for transient failures
- Rollback on partial failure

### Result Processor

Transforms tool outputs into user-facing responses.

- Output validation against expected schema
- Result formatting and summarization
- Error translation to user-friendly messages
- Citation of tool sources

## Available Tools (Example)

```json
{
  "tools": [
    {
      "name": "web_search",
      "description": "Search the web for current information",
      "parameters": {
        "query": { "type": "string", "required": true },
        "num_results": { "type": "integer", "default": 5 }
      }
    },
    {
      "name": "code_interpreter",
      "description": "Execute code in a sandboxed environment",
      "parameters": {
        "language": { "type": "string", "enum": ["python", "javascript"] },
        "code": { "type": "string", "required": true }
      }
    },
    {
      "name": "file_manager",
      "description": "Read, write, and manage files",
      "parameters": {
        "action": { "type": "string", "enum": ["read", "write", "list", "delete"] },
        "path": { "type": "string", "required": true },
        "content": { "type": "string" }
      }
    }
  ]
}
```

## Evaluation

| Metric                    | Target   | Method                        |
|---------------------------|----------|-------------------------------|
| Tool Selection Accuracy   | ≥ 0.90   | Intent-to-tool matching test  |
| Parameter Extraction      | ≥ 0.85   | NER and slot filling eval     |
| Execution Success Rate    | ≥ 0.95   | Production monitoring         |
| End-to-end Latency (p95)  | < 5s     | Production monitoring         |
| Error Recovery Rate       | ≥ 0.80   | Fault injection testing       |

## Quality Checklist

- [ ] Tool schemas are complete and accurate
- [ ] Authentication is handled securely
- [ ] Tool selection reasoning is logged
- [ ] Execution sandbox enforces resource limits
- [ ] Partial failures are handled gracefully
- [ ] Tool outputs are validated before use
- [ ] Cost tracking is per-tool-call

## Failure Modes

| Failure                   | Detection              | Mitigation                        |
|---------------------------|------------------------|------------------------------------|
| Tool unavailability       | Health check           | Fallback tools + user notification |
| Parameter validation fail | Schema validation      | Clarification prompt               |
| Execution timeout         | Timeout monitoring     | Kill + retry with backoff          |
| Output exceeds limits     | Size validation        | Truncation + summary               |
| Tool returns error        | Error code checking    | Error translation + retry logic    |

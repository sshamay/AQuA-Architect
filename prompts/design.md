# Design Prompts

Structured prompts for the Design Mode of AQuA-Architect.

---

## System Design Prompt

```
You are an AI Quality Engineering Architect designing a production-ready system.

Given the following requirements:

{requirements}

Design a complete system architecture that addresses:

1. **Component Identification**
   - List all required components with single responsibilities
   - Define the data flow between components
   - Identify external dependencies and services

2. **Interface Design**
   - Define APIs and data contracts for each component
   - Specify input/output types for all interfaces
   - Document error response formats

3. **Failure Mode Analysis**
   - Identify potential failure modes for each component
   - Define detection mechanisms for each failure
   - Specify mitigation strategies and recovery procedures

4. **Observability Plan**
   - Define telemetry points for each component
   - Specify log structure and severity levels
   - Identify key metrics and their thresholds
   - Plan distributed tracing strategy

5. **Evaluation Strategy**
   - Define success metrics with targets
   - Specify evaluation dataset requirements
   - Plan automated evaluation pipeline

Constraints:
{constraints}

Output a structured architecture document following the patterns in architecture.md.
```

## Requirements Analysis Prompt

```
Analyze the following requirements and extract:

Requirements:
{raw_requirements}

Produce:

1. **Functional Requirements**
   - Numbered list of capabilities the system must provide
   - Priority level for each (P0, P1, P2)

2. **Non-Functional Requirements**
   - Performance targets (latency, throughput)
   - Reliability targets (availability, error rates)
   - Scalability requirements
   - Security requirements

3. **Constraints**
   - Technical constraints
   - Business constraints
   - Regulatory constraints

4. **Ambiguities and Questions**
   - List unclear requirements needing clarification
   - Propose reasonable defaults for each

5. **Risk Assessment**
   - Technical risks with likelihood and impact
   - Mitigation strategies for each risk
```

## Component Design Prompt

```
Design the component: {component_name}

Context:
{system_context}

Requirements for this component:
{component_requirements}

Provide:

1. **Responsibility Statement** — Single sentence describing the component's purpose
2. **Interface Specification**
   - All public methods/functions with signatures
   - Input validation rules
   - Output guarantees
   - Error types and conditions

3. **Internal Architecture**
   - Key data structures
   - Core algorithms or logic
   - State management approach

4. **Dependencies**
   - Required external services
   - Required internal components
   - Configuration requirements

5. **Observability**
   - Events to emit
   - Metrics to track
   - Logs to generate

6. **Test Strategy**
   - Unit test cases
   - Integration test cases
   - Edge cases to cover
```

## Prompt Template Design Prompt

```
Design a prompt template for the following use case:

Use Case: {use_case}
Target Model: {model}
Expected Input: {input_description}
Expected Output: {output_description}

Create:

1. **System Prompt**
   - Role definition
   - Behavioral guidelines
   - Constraints and guardrails

2. **User Prompt Template**
   - Variable slots with types
   - Example values for each slot
   - Input validation rules

3. **Output Schema**
   - Expected structure
   - Required fields
   - Validation rules

4. **Safety Considerations**
   - Potential misuse scenarios
   - Mitigation instructions in prompt
   - Output filtering requirements

5. **Evaluation Criteria**
   - Quality metrics for this prompt
   - Example inputs and expected outputs
   - Failure cases to test
```

# Review Prompts

Structured prompts for the Review Mode of AQuA-Architect.

---

## Code Review Prompt

```
You are reviewing code for quality, reliability, and adherence to architecture.

Implementation:
{implementation_code}

Architecture Document:
{architecture_document}

Review for:

1. **Correctness**
   - Logic errors
   - Off-by-one errors
   - Null/undefined handling
   - Race conditions

2. **Architecture Compliance**
   - Does it match the architecture document?
   - Are interfaces correctly implemented?
   - Is separation of concerns maintained?

3. **Error Handling**
   - Are all failure modes handled?
   - Are errors properly classified and propagated?
   - Is recovery logic correct?

4. **Security**
   - Input validation
   - Output sanitization
   - Secret handling
   - Access control

5. **Performance**
   - Unnecessary allocations
   - Algorithmic inefficiencies
   - N+1 query patterns
   - Caching opportunities

For each finding:
- **Severity:** Critical / High / Medium / Low / Info
- **Location:** File and line number
- **Issue:** Clear description
- **Recommendation:** Specific fix

Provide a summary score using the rubric in scoring.md.
```

## Architecture Review Prompt

```
Review the following system architecture:

{architecture_document}

Evaluate:

1. **Completeness**
   - Are all requirements addressed?
   - Are all interfaces defined?
   - Is observability planned?

2. **Reliability**
   - Are failure modes identified?
   - Are recovery strategies defined?
   - Is the system designed for graceful degradation?

3. **Scalability**
   - What are the bottlenecks?
   - Can components scale independently?
   - Are resource limits documented?

4. **Maintainability**
   - Is the system easy to understand?
   - Are components loosely coupled?
   - Is documentation sufficient?

5. **Security**
   - Attack surface analysis
   - Data protection measures
   - Access control design

Provide specific recommendations with priority and effort estimates.
```

## Prompt Quality Review Prompt

```
Review the following AI prompt for quality:

Prompt:
{prompt_template}

Evaluate:

1. **Clarity**
   - Is the role clearly defined?
   - Are instructions unambiguous?
   - Is the output format specified?

2. **Completeness**
   - Are all requirements covered?
   - Are edge cases addressed?
   - Are constraints documented?

3. **Safety**
   - Are guardrails in place?
   - Is misuse prevented?
   - Are output filters specified?

4. **Consistency**
   - Does it produce consistent outputs?
   - Are examples representative?
   - Is the template robust?

5. **Effectiveness**
   - Does it achieve the intended goal?
   - Are quality metrics defined?
   - Is evaluation possible?

Provide specific improvements with priority.
```

## Security Review Prompt

```
Perform a security review of:

{implementation_or_architecture}

Check for:

1. **Input Validation**
   - SQL injection
   - Command injection
   - XSS vulnerabilities
   - Path traversal

2. **Authentication & Authorization**
   - Missing auth checks
   - Privilege escalation
   - Session management

3. **Data Protection**
   - Sensitive data exposure
   - Insecure storage
   - Inadequate encryption

4. **Configuration**
   - Default credentials
   - Debug mode in production
   - Verbose error messages

5. **Dependencies**
   - Known vulnerabilities
   - Outdated packages
   - Supply chain risks

For each finding:
- **CVSS Score** (estimated)
- **Attack Vector**
- **Impact**
- **Remediation** with code example

Rate overall security posture: Critical / High / Medium / Low Risk
```

## Observability Review Prompt

```
Review the observability implementation:

{implementation_code_or_config}

Evaluate:

1. **Logging**
   - Is structured logging used?
   - Are all levels used appropriately?
   - Is correlation ID propagation complete?
   - Are sensitive values excluded?

2. **Metrics**
   - Are key operations instrumented?
   - Are metrics named consistently?
   - Are histograms used for latency?
   - Are business metrics tracked?

3. **Tracing**
   - Are critical paths traced?
   - Are spans correctly parented?
   - Are errors annotated?
   - Is context propagation working?

4. **Alerting**
   - Are alerts actionable?
   - Are thresholds appropriate?
   - Is alert fatigue managed?
   - Are runbooks linked?

5. **Dashboards**
   - Are key metrics visible?
   - Is drill-down possible?
   - Are error budgets tracked?

Provide a gap analysis and improvement plan.
```

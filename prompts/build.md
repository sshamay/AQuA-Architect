# Build Prompts

Structured prompts for the Build Mode of AQuA-Architect.

---

## Implementation Prompt

```
You are implementing a component for a production-ready AI system.

Architecture Document:
{architecture_document}

Component to implement: {component_name}

Requirements:
1. Follow the architecture document exactly
2. Include comprehensive error handling
3. Add structured logging at all levels
4. Implement input validation
5. Write inline documentation for public APIs

Language/Framework: {language_framework}
Dependencies allowed: {allowed_dependencies}

Output:
- Complete implementation code
- Configuration files
- Unit test file
- Brief setup instructions
```

## Test Generation Prompt

```
Generate comprehensive tests for the following implementation:

{implementation_code}

Generate:

1. **Unit Tests**
   - Test each public method/function
   - Cover happy path, edge cases, and error cases
   - Use descriptive test names
   - Include setup and teardown

2. **Integration Tests**
   - Test component interactions
   - Mock external dependencies
   - Test error propagation

3. **Test Data**
   - Valid input fixtures
   - Boundary value fixtures
   - Invalid input fixtures

Follow testing best practices:
- Each test should be independent
- Arrange-Act-Assert structure
- Clear assertions with messages
- No test interdependencies
```

## Error Handling Prompt

```
Review and improve error handling for:

{implementation_code}

Context:
- Component purpose: {purpose}
- External dependencies: {dependencies}
- Failure modes identified: {failure_modes}

For each potential error:

1. **Detection** — How to detect the error
2. **Classification** — Transient vs permanent
3. **Recovery** — Retry, fallback, or fail gracefully
4. **Logging** — What to log and at what level
5. **User Impact** — How to communicate to users

Output improved code with comprehensive error handling.
```

## Observability Instrumentation Prompt

```
Add observability instrumentation to:

{implementation_code}

Instrument:

1. **Structured Logging**
   - Log entry/exit of major operations
   - Log errors with context
   - Include correlation IDs
   - Use structured JSON format

2. **Metrics**
   - Operation counters
   - Latency histograms
   - Error rate gauges
   - Business-specific metrics

3. **Tracing**
   - Span creation for key operations
   - Parent-child relationships
   - Error annotation on spans

4. **Events**
   - Business events for auditing
   - System events for monitoring

Output the instrumented code with all observability hooks.
```

## Refactoring Prompt

```
Refactor the following code for quality and reliability:

{implementation_code}

Focus on:
1. **SOLID Principles** — Single responsibility, dependency inversion
2. **Error Handling** — Comprehensive and consistent
3. **Type Safety** — Explicit types, no implicit any
4. **Code Clarity** — Naming, structure, readability
5. **Performance** — Identify and fix obvious inefficiencies

Preserve all existing functionality. Output the refactored code.
```

## Documentation Prompt

```
Generate documentation for:

{implementation_code}

Produce:

1. **Component Overview** — What it does and why it exists
2. **API Reference** — All public interfaces with descriptions
3. **Configuration** — All configurable options
4. **Usage Examples** — Common usage patterns
5. **Error Reference** — All possible errors and their meanings
6. **Architecture Notes** — Design decisions and trade-offs

Format as clear, developer-friendly documentation.
```

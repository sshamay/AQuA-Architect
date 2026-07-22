# Code Review Example

An AI-assisted code review system that analyzes pull requests for quality, security, and best practices.

## Architecture

```
PR Webhook → Event Handler → Diff Analyzer → Review Engine → Comment Formatter → PR Commenter
                                    ↓               ↓
                              AST Parser      Rule Engine
                              (Language)      (Custom Rules)
```

## Components

### Event Handler

Receives and processes PR events from version control.

- Webhook validation and authentication
- Event deduplication
- PR metadata extraction
- Review triggering logic

### Diff Analyzer

Parses and understands code changes.

- Diff parsing and normalization
- AST-level analysis (not just text)
- Language detection and routing
- Change classification (feature, bugfix, refactor)

### Review Engine

Applies rules and generates review feedback.

- Static analysis integration
- Custom rule evaluation
- Pattern detection (anti-patterns, smells)
- Severity classification (error, warning, info)

### Rule Engine

Manages review rules and their configurations.

- Rule categories: correctness, security, performance, style
- Rule severity and confidence levels
- Per-language rule sets
- Custom rule support

### Comment Formatter

Structures feedback for developers.

- Inline code comments
- Summary reports
- Severity-based grouping
- Actionable suggestions with code examples

## Review Categories

### Correctness
- Potential null pointer dereferences
- Unhandled error conditions
- Off-by-one errors
- Resource leaks

### Security
- SQL injection vulnerabilities
- XSS potential
- Hardcoded secrets
- Insecure dependencies

### Performance
- Unnecessary computations
- Memory allocation patterns
- Database query optimization
- Caching opportunities

### Maintainability
- Code complexity (cyclomatic)
- Function/method length
- Naming conventions
- Documentation coverage

## Evaluation

| Metric                    | Target   | Method                        |
|---------------------------|----------|-------------------------------|
| True Positive Rate        | ≥ 0.85   | Human review of flagged issues|
| False Positive Rate       | ≤ 0.15   | Human review of flagged issues|
| Issue Detection Recall    | ≥ 0.75   | Known bug injection testing   |
| Review Latency            | < 60s    | Production monitoring         |
| Developer Satisfaction    | ≥ 4.0/5  | Feedback survey               |

## Quality Checklist

- [ ] All review rules are documented
- [ ] False positive rate is within acceptable bounds
- [ ] Language-specific rules are properly configured
- [ ] Review comments include fix suggestions
- [ ] Review results are logged and trackable
- [ ] Performance impact on CI is minimal
- [ ] Critical issues block PR merge

## Failure Modes

| Failure                    | Detection              | Mitigation                        |
|----------------------------|------------------------|------------------------------------|
| Parser crash on new syntax | Error logging          | Graceful skip + report             |
| Excessive false positives  | Feedback tracking      | Rule tuning + confidence thresholds|
| Review timeout             | Timeout monitoring     | Partial review + retry             |
| Rule conflict              | Priority system        | Precedence-based resolution        |
| Large PR performance       | Diff size monitoring   | Chunked review + prioritization    |

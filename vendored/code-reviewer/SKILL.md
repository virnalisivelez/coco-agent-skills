---
name: code-reviewer
description: Perform thorough code reviews with structured feedback. Use when reviewing pull requests, auditing code quality, checking for security issues, or evaluating architectural decisions. Provides categorized feedback (critical, important, minor) with actionable suggestions.
license: Apache-2.0
metadata:
  original-author: terminal-skills
  original-repo: TerminalSkills/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# Code Reviewer

Perform structured, thorough code reviews that catch real issues and provide actionable feedback. Reviews are categorized by severity and organized for easy consumption.

## When to Use

- User wants a code review of a file, module, or pull request
- User asks to audit code quality or check for issues
- User wants feedback on architectural decisions
- User needs a pre-merge checklist before shipping

## Review Process

Follow this sequence for every review:

### 1. Understand Context

Before reading code, establish:
- What does this code do? (feature, bugfix, refactor)
- What language/framework is it written in?
- Are there tests included?
- Is there a PR description or issue reference?

### 2. First Pass: Architecture and Design

Read the entire changeset once for structural understanding:
- Does the overall approach make sense for the problem?
- Is the code in the right place (file, module, layer)?
- Are responsibilities properly separated?
- Does it follow existing patterns in the codebase?
- Are there simpler ways to achieve the same result?

### 3. Second Pass: Line-by-Line Analysis

Walk through each file, checking for issues across these categories:

#### Security
- Input validation and sanitization
- SQL injection, XSS, CSRF exposure
- Secrets or credentials in code
- Unsafe deserialization
- Improper authentication/authorization checks
- File path traversal risks

#### Performance
- N+1 queries or unnecessary database calls
- Missing pagination on list endpoints
- Unbounded loops or recursion
- Large objects held in memory unnecessarily
- Missing caching where appropriate
- Expensive operations inside loops

#### Correctness
- Edge cases: null, empty, zero, negative, very large values
- Off-by-one errors in loops and slices
- Race conditions in concurrent code
- Unchecked return values or error codes
- Type mismatches or unsafe casts
- Timezone handling in date/time operations

#### Maintainability
- Function/method length (flag anything over 50 lines)
- Cyclomatic complexity (deeply nested conditionals)
- Dead code or commented-out blocks
- Magic numbers without named constants
- Duplicated logic that should be extracted
- Missing or misleading comments

#### Naming and Style
- Variable and function names are descriptive and consistent
- Follows the project's naming conventions
- Consistent formatting (indentation, spacing, line length)
- Imports organized and unused imports removed

#### Error Handling
- All error paths are handled, not silently swallowed
- Error messages are helpful for debugging
- Errors propagate correctly (thrown, returned, logged)
- Cleanup happens in finally/defer blocks where needed
- User-facing errors are safe (no stack traces or internals)

#### Testing
- Are new code paths covered by tests?
- Do tests verify behavior, not implementation?
- Are edge cases tested?
- Are error paths tested?
- Do tests have clear assertions and failure messages?
- Is there integration test coverage for cross-module changes?

### 4. Synthesize Findings

After both passes, compile all findings into the structured output format below.

## Output Format

Present findings in three severity tiers, then a summary:

```markdown
## Code Review: [filename or PR title]

### Critical (must fix before merge)

Issues that will cause bugs, security vulnerabilities, data loss, or outages.

- **[Category] [File:Line]**: Description of the issue.
  - *Why it matters:* Explanation of the risk.
  - *Suggested fix:* Concrete code or approach to resolve.

### Important (strongly recommended)

Issues that degrade quality, performance, or maintainability significantly.

- **[Category] [File:Line]**: Description of the issue.
  - *Suggested fix:* Concrete code or approach to resolve.

### Minor (nice to have)

Style, naming, or minor improvements that are not blocking.

- **[Category] [File:Line]**: Description of the issue.
  - *Suggested fix:* Brief recommendation.

### Summary

- **Verdict:** Approve / Request Changes / Needs Discussion
- **Critical issues:** N
- **Important issues:** N
- **Minor issues:** N
- **Strengths:** What the code does well (always include at least one).
- **Testing:** Assessment of test coverage adequacy.
```

## Review Principles

1. **Be specific.** Point to exact lines. Do not say "this could be better" without saying how.
2. **Suggest, do not demand.** Use "Consider..." or "Suggested fix:" rather than "You must..."
3. **Explain the why.** A reviewer who only says "change this" teaches nothing.
4. **Acknowledge good work.** If the code is well-structured, say so. Reviews are not only for finding faults.
5. **Prioritize ruthlessly.** A review with 3 critical findings is more useful than one with 40 nitpicks.
6. **Stay in scope.** Review the code that changed, not the entire file history.
7. **No style wars.** If the project has a formatter (Prettier, Black, gofmt), trust it.

## Language-Specific Checks

### Python
- Type hints on function signatures
- f-strings preferred over `.format()` or `%`
- Context managers (`with`) for file/connection handling
- `pathlib.Path` over `os.path` for path manipulation
- Avoid mutable default arguments (`def f(x=[])`)

### JavaScript/TypeScript
- Strict equality (`===`) over loose (`==`)
- Proper async/await error handling (try/catch or .catch)
- No `any` type in TypeScript without justification
- Event listener cleanup in component unmount
- Avoid `var`; use `const` by default, `let` when needed

### SQL
- Parameterized queries (never string concatenation)
- Explicit column lists (no `SELECT *` in production)
- Index coverage for WHERE/JOIN columns
- Transaction boundaries for multi-statement operations

### Go
- Error values checked immediately after function calls
- `defer` for cleanup (file close, mutex unlock)
- Context propagation through function chains
- No goroutine leaks (ensure goroutines can exit)

## Common Anti-Patterns to Flag

| Anti-Pattern | Why It Matters |
|---|---|
| God function (100+ lines) | Impossible to test, hard to debug |
| Boolean parameters | Unclear call sites; use enums or options |
| Deep nesting (4+ levels) | Extract to named functions |
| Silent catch blocks | Bugs become invisible |
| Hardcoded URLs/credentials | Security risk, deployment inflexibility |
| TODO without issue link | TODOs without tracking are never resolved |
| Copy-paste duplication | Maintenance burden multiplies |
| Over-engineering | Abstractions for one use case add complexity |

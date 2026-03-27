---
name: coco-multi-agent-system
description: Build production-style multi-agent systems that plan, implement, test, and evaluate autonomously. Use when building orchestrated agent pipelines, breaking product ideas into milestones, or creating systems with planning, execution, testing, and evaluation layers.
license: Apache-2.0
metadata:
  author: good-stories-llc
  version: "1.0"
  brand: COCO
---

# Multi-Agent System Builder

Build production-grade multi-agent systems with orchestration, planning, implementation, testing, and evaluation layers. Transform product ideas into working prototypes through autonomous agent collaboration.

## When to Use

- User wants to build a multi-agent pipeline for a product or feature
- User needs to break a complex project into milestones with autonomous execution
- User asks for an agent architecture with planning, coding, testing, and review
- User wants a system that can self-correct through iterative refinement

## Architecture Overview

```
                    +-------------------+
                    |   Orchestrator    |
                    | (milestone loop)  |
                    +--------+----------+
                             |
         +-------------------+-------------------+
         |         |         |         |         |
    +----v---+ +---v----+ +-v------+ +v-------+ +v----------+
    |  Spec  | |  Arch  | | Impl  | | Test   | | Evaluate  |
    | Agent  | | Agent  | | Agent | | Agent  | | Agent     |
    +--------+ +--------+ +-------+ +--------+ +-----------+
```

Each agent has a single responsibility. The Orchestrator sequences them, handles retries, and decides when a milestone is complete.

## The Orchestrator

The Orchestrator is the control loop. It manages the pipeline from start to finish.

### Responsibilities

1. **Parse the product idea** into discrete milestones (MVP, v1, v2)
2. **Sequence agent calls** for each milestone: Spec -> Arch -> Impl -> Test -> Evaluate
3. **Track progress** with a status object per milestone
4. **Handle failures** with configurable retry logic
5. **Decide completion** based on evaluation scores

### Orchestrator Logic

```python
class Orchestrator:
    def __init__(self, product_idea: str, max_retries: int = 3):
        self.product_idea = product_idea
        self.max_retries = max_retries
        self.milestones = []
        self.results = {}

    def run(self):
        # Phase 1: Break idea into milestones
        self.milestones = self.plan_milestones(self.product_idea)

        for milestone in self.milestones:
            self.execute_milestone(milestone)

        return self.compile_final_report()

    def execute_milestone(self, milestone: dict):
        retries = 0
        while retries < self.max_retries:
            spec = SpecAgent().run(milestone)
            arch = ArchAgent().run(spec)
            impl = ImplAgent().run(arch)
            test_results = TestAgent().run(impl)
            evaluation = EvalAgent().run(impl, test_results)

            if evaluation["pass"]:
                self.results[milestone["id"]] = {
                    "status": "complete",
                    "code": impl,
                    "tests": test_results,
                    "score": evaluation["score"],
                }
                return

            retries += 1
            milestone["feedback"] = evaluation["feedback"]

        self.results[milestone["id"]] = {"status": "failed_after_retries"}
```

### Milestone Structure

Each milestone is a self-contained deliverable:

```python
{
    "id": "milestone-1-mvp",
    "title": "Core API with authentication",
    "description": "REST API with user registration, login, and JWT auth",
    "acceptance_criteria": [
        "POST /register creates a user",
        "POST /login returns a JWT",
        "Protected endpoints reject invalid tokens",
    ],
    "dependencies": [],
    "feedback": None,  # Populated on retry
}
```

## Specification Agent

The Spec Agent transforms a milestone into a detailed specification.

### Outputs

1. **Product Requirements Document (PRD):** What the feature does, who it serves, success criteria
2. **User Stories:** As a [role], I want [action], so that [benefit]
3. **API Contracts:** Endpoint definitions with request/response schemas
4. **Data Model:** Entities, fields, relationships, constraints
5. **Non-Functional Requirements:** Performance targets, security requirements, scalability needs

### Spec Agent Prompt Pattern

```
You are a product specification agent. Given a milestone description
and acceptance criteria, produce a detailed specification.

Milestone: {milestone.title}
Description: {milestone.description}
Acceptance Criteria: {milestone.acceptance_criteria}
Previous Feedback: {milestone.feedback or "None (first iteration)"}

Output a specification document with these sections:
1. PRD (2-3 paragraphs)
2. User Stories (3-5 stories in standard format)
3. API Contracts (endpoint, method, request body, response body, status codes)
4. Data Model (entities with fields and types)
5. Non-Functional Requirements (performance, security, scalability)
```

## Architecture Agent

The Arch Agent designs the system structure from the specification.

### Outputs

1. **Folder Structure:** Complete directory layout with file purposes
2. **Schema Definitions:** Database tables, API schemas, type definitions
3. **Service Boundaries:** Which modules own which responsibilities
4. **Dependency Map:** Internal module dependencies and external packages
5. **Technology Choices:** Framework, database, libraries with rationale

### Architecture Template

```
project/
  src/
    api/
      routes.py          # Endpoint definitions
      middleware.py       # Auth, rate limiting, error handling
      schemas.py          # Request/response Pydantic models
    core/
      services.py         # Business logic
      models.py           # Database models
      exceptions.py       # Custom exception classes
    db/
      connection.py       # Database connection management
      migrations/         # Schema migration files
  tests/
    test_api.py           # Endpoint integration tests
    test_services.py      # Business logic unit tests
    conftest.py           # Shared fixtures
  config.py               # Environment and app configuration
  main.py                 # Application entry point
  requirements.txt        # Dependencies
```

## Implementation Agent

The Impl Agent writes production-quality code from the architecture.

### Code Standards

1. **Modular:** Each file has a single responsibility. No file exceeds 300 lines.
2. **Typed:** Full type hints on all function signatures (Python) or TypeScript types (JS).
3. **Documented:** Docstrings on every public function explaining what it does, parameters, and return value.
4. **Error-Handled:** Every external call (database, API, file I/O) has error handling.
5. **Configurable:** No hardcoded values. Use environment variables or config files.
6. **Idempotent:** Operations that can be retried safely are designed that way.

### Implementation Checklist

- [ ] All API endpoints match the spec contracts exactly
- [ ] Input validation on every endpoint (type, range, format)
- [ ] Authentication and authorization checks on protected routes
- [ ] Database queries use parameterized statements
- [ ] External API calls have timeouts and retries
- [ ] Logging on key operations (startup, errors, critical business events)
- [ ] Configuration loaded from environment variables

## Testing Agent

The Test Agent generates and runs tests against the implementation.

### Test Layers

1. **Unit Tests:** Test individual functions and methods in isolation
   - Mock external dependencies
   - Cover happy path, edge cases, error paths
   - Target: 80%+ line coverage

2. **Integration Tests:** Test component interactions
   - API endpoint tests with a test client
   - Database tests with a test database
   - End-to-end request/response validation

3. **Contract Tests:** Verify implementation matches the spec
   - Every API contract from the Spec Agent has a corresponding test
   - Response schemas match exactly
   - Status codes are correct for all scenarios

### Test Report Format

```python
{
    "total": 24,
    "passed": 22,
    "failed": 2,
    "coverage": 87.3,
    "failures": [
        {
            "test": "test_login_with_expired_token",
            "error": "Expected 401, got 500",
            "file": "tests/test_api.py:45",
        },
    ],
}
```

## Evaluation Agent

The Eval Agent scores the implementation and decides if the milestone passes.

### Evaluation Criteria

| Dimension | Weight | What It Measures |
|---|---|---|
| Correctness | 30% | All tests pass, acceptance criteria met |
| Code Quality | 20% | Readable, modular, well-named, documented |
| Security | 20% | Auth, input validation, no secrets, injection-safe |
| Maintainability | 15% | Easy to modify, extend, and debug |
| Scalability | 15% | Handles growth in data and traffic |

### Scoring

Each dimension is scored 1-5:
- **5:** Production-ready, no issues
- **4:** Minor improvements possible, safe to ship
- **3:** Functional but needs refinement before production
- **2:** Significant issues that must be addressed
- **1:** Fundamentally broken or missing

**Pass threshold:** Weighted average >= 3.5 AND no dimension below 2.

### Evaluation Output

```python
{
    "pass": True,
    "score": 4.1,
    "breakdown": {
        "correctness": 5,
        "code_quality": 4,
        "security": 4,
        "maintainability": 3,
        "scalability": 4,
    },
    "feedback": [
        "Add input length validation on the /register endpoint",
        "Extract database connection string to environment variable",
        "Add index on users.email for login query performance",
    ],
}
```

## Iterative Refinement Loop

When a milestone fails evaluation:

1. The Orchestrator passes `evaluation.feedback` back into the milestone
2. The Spec Agent reviews and updates the spec with the feedback
3. The Arch Agent adjusts the architecture if needed
4. The Impl Agent applies targeted fixes (not a full rewrite)
5. The Test Agent re-runs all tests
6. The Eval Agent re-evaluates

The Impl Agent receives both the previous code and the feedback, so it makes incremental corrections rather than starting from scratch.

## Observability and Logging

Every agent call should be logged for debugging and auditability:

```python
import logging
from datetime import datetime, timezone

logger = logging.getLogger("multi_agent")

def log_agent_call(agent_name: str, milestone_id: str, input_summary: str, output_summary: str, duration_ms: int):
    logger.info(
        "agent_call",
        extra={
            "agent": agent_name,
            "milestone": milestone_id,
            "input": input_summary[:200],
            "output": output_summary[:200],
            "duration_ms": duration_ms,
            "timestamp": datetime.now(timezone.utc).isoformat(),
        },
    )
```

## Final Output Requirements

When the Orchestrator completes all milestones, deliver:

1. **Working prototype** with all code files
2. **Test results** with pass/fail and coverage
3. **Risk assessment** identifying known limitations and failure modes
4. **Tech debt report** listing shortcuts taken and future improvements
5. **Next iteration roadmap** with prioritized enhancements for the next cycle

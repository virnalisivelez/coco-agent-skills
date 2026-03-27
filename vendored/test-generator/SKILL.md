---
name: test-generator
description: Auto-generate comprehensive test suites for any codebase. Use when writing unit tests, integration tests, or adding test coverage to existing code. Supports Python pytest, JavaScript Jest, and other frameworks.
license: Apache-2.0
metadata:
  original-author: terminal-skills
  original-repo: TerminalSkills/skills
  vendored-by: good-stories-llc
  version: "1.0"
---

# Test Generator

Generate comprehensive, well-structured test suites for any codebase. Covers unit tests, integration tests, edge cases, and error paths with framework-specific patterns.

## When to Use

- User asks to write tests for existing code
- User wants to increase test coverage
- User needs edge case identification for a function or module
- User wants to set up a testing framework from scratch

## Test Generation Methodology

### Step 1: Analyze the Code Under Test

Before writing any test, answer these questions:

1. **What are the inputs?** Parameters, environment variables, external data
2. **What are the outputs?** Return values, side effects, exceptions, state changes
3. **What are the dependencies?** Database, APIs, file system, time, randomness
4. **What are the branches?** Every `if`, `switch`, `try/catch`, ternary, and guard clause
5. **What could go wrong?** Invalid inputs, network failures, empty data, race conditions

### Step 2: Identify Test Cases

For each function or method, generate test cases in this order:

#### Happy Path (the normal case works)
- Typical valid input produces expected output
- Multiple valid input combinations if the function is polymorphic

#### Boundary Values
- Zero, one, max value for numeric inputs
- Empty string, single character, max-length string
- Empty list/array, single element, large collection
- First and last items in ranges
- Date boundaries (midnight, end of month, leap year, DST transitions)

#### Edge Cases
- Null/None/undefined inputs
- Negative numbers where only positives are expected
- Unicode, special characters, whitespace-only strings
- Very large inputs (test for performance, not just correctness)
- Concurrent access if the code is shared across threads

#### Error Paths
- Invalid input types (string where number expected)
- Missing required fields
- Network/database failures (via mocking)
- Permission denied, file not found
- Timeout scenarios

#### State Transitions (if stateful)
- Initial state is correct
- Each valid transition produces the right state
- Invalid transitions are rejected

### Step 3: Write Tests

Follow the AAA pattern for every test: Arrange, Act, Assert.

```
Arrange: Set up the inputs, mocks, and expected outputs.
Act:     Call the function under test.
Assert:  Verify the output matches expectations.
```

## Python (pytest) Patterns

### Basic Test Structure

```python
import pytest
from mymodule import calculate_total

class TestCalculateTotal:
    """Tests for the calculate_total function."""

    def test_returns_sum_of_items(self):
        items = [{"price": 10.0, "qty": 2}, {"price": 5.0, "qty": 1}]
        result = calculate_total(items)
        assert result == 25.0

    def test_returns_zero_for_empty_list(self):
        assert calculate_total([]) == 0.0

    def test_raises_on_negative_price(self):
        items = [{"price": -5.0, "qty": 1}]
        with pytest.raises(ValueError, match="Price cannot be negative"):
            calculate_total(items)
```

### Parametrized Tests

Use `@pytest.mark.parametrize` when testing the same logic with multiple inputs:

```python
@pytest.mark.parametrize("input_val, expected", [
    (0, "zero"),
    (1, "one"),
    (42, "forty-two"),
    (-1, "negative"),
])
def test_number_to_word(input_val, expected):
    assert number_to_word(input_val) == expected
```

### Fixtures

Use fixtures for shared setup. Keep them close to the tests that use them:

```python
@pytest.fixture
def sample_user():
    return User(name="Test User", email="test@example.com", role="admin")

@pytest.fixture
def db_session(tmp_path):
    """Create a temporary SQLite database for testing."""
    db_path = tmp_path / "test.db"
    engine = create_engine(f"sqlite:///{db_path}")
    Base.metadata.create_all(engine)
    session = Session(engine)
    yield session
    session.close()
```

### Mocking External Dependencies

```python
from unittest.mock import patch, MagicMock

def test_fetch_user_calls_api_with_correct_url():
    mock_response = MagicMock()
    mock_response.json.return_value = {"id": 1, "name": "Alice"}
    mock_response.status_code = 200

    with patch("mymodule.requests.get", return_value=mock_response) as mock_get:
        result = fetch_user(1)
        mock_get.assert_called_once_with("https://api.example.com/users/1")
        assert result["name"] == "Alice"

def test_fetch_user_handles_api_failure():
    with patch("mymodule.requests.get", side_effect=ConnectionError("timeout")):
        with pytest.raises(ServiceUnavailableError):
            fetch_user(1)
```

### Async Tests

```python
import pytest

@pytest.mark.asyncio
async def test_async_fetch():
    result = await async_fetch_data("test-id")
    assert result.status == "ok"
```

## JavaScript (Jest) Patterns

### Basic Test Structure

```javascript
const { calculateTotal } = require('./cart');

describe('calculateTotal', () => {
  it('returns the sum of item prices times quantities', () => {
    const items = [
      { price: 10.0, qty: 2 },
      { price: 5.0, qty: 1 },
    ];
    expect(calculateTotal(items)).toBe(25.0);
  });

  it('returns 0 for an empty array', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('throws on negative price', () => {
    const items = [{ price: -5, qty: 1 }];
    expect(() => calculateTotal(items)).toThrow('Price cannot be negative');
  });
});
```

### Mocking

```javascript
// Mock a module
jest.mock('./api', () => ({
  fetchUser: jest.fn(),
}));

const { fetchUser } = require('./api');
const { getUserName } = require('./user-service');

describe('getUserName', () => {
  it('returns the user name from the API', async () => {
    fetchUser.mockResolvedValue({ id: 1, name: 'Alice' });
    const name = await getUserName(1);
    expect(name).toBe('Alice');
    expect(fetchUser).toHaveBeenCalledWith(1);
  });

  it('throws when the API fails', async () => {
    fetchUser.mockRejectedValue(new Error('Network error'));
    await expect(getUserName(1)).rejects.toThrow('Network error');
  });
});
```

### Setup and Teardown

```javascript
describe('DatabaseService', () => {
  let db;

  beforeEach(async () => {
    db = await createTestDatabase();
  });

  afterEach(async () => {
    await db.destroy();
  });

  it('inserts and retrieves a record', async () => {
    await db.insert('users', { name: 'Alice' });
    const user = await db.findOne('users', { name: 'Alice' });
    expect(user).toBeDefined();
    expect(user.name).toBe('Alice');
  });
});
```

## Test Quality Guidelines

### Naming

Test names should describe the behavior, not the implementation:

- Good: `test_returns_empty_list_when_no_results_match`
- Bad: `test_filter_function`
- Good: `it('disables the submit button when the form is invalid')`
- Bad: `it('works')`

### Independence

- Each test must be independent. No test should depend on another test's side effects.
- Use fresh fixtures/setup for each test.
- Never rely on test execution order.

### Assertions

- One logical assertion per test (multiple `assert` calls are fine if they verify one behavior)
- Use specific assertions: `assertEqual`, `assertIn`, `toHaveBeenCalledWith`
- Avoid `assertTrue(result)` -- use `assertEqual(result, expected_value)`

### What NOT to Test

- Framework internals (trust that React renders, Express routes, etc.)
- Trivial getters/setters with no logic
- Third-party library behavior
- Implementation details (private methods, internal state)

### Test Coverage Targets

- Aim for 80%+ line coverage as a baseline
- 100% coverage on critical paths: authentication, payments, data mutations
- Coverage is a guide, not a goal -- untested edge cases matter more than a number

## Integration Test Patterns

For code that crosses boundaries (API endpoints, database operations):

```python
# FastAPI integration test
from fastapi.testclient import TestClient
from myapp import app

client = TestClient(app)

def test_create_and_retrieve_user():
    # Create
    response = client.post("/users", json={"name": "Alice", "email": "a@b.com"})
    assert response.status_code == 201
    user_id = response.json()["id"]

    # Retrieve
    response = client.get(f"/users/{user_id}")
    assert response.status_code == 200
    assert response.json()["name"] == "Alice"
```

## Output Expectations

When generating tests:

1. Deliver complete, runnable test files with all imports
2. Group tests logically in classes or `describe` blocks
3. Include setup/teardown for any shared state
4. Cover happy path, boundaries, edge cases, and error paths
5. Mock external dependencies -- tests must not require network or database
6. Add brief comments explaining non-obvious test rationale

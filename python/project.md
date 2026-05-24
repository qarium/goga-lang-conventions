# General Conventions

Mandatory rules for all Python code in this project.

---

# Development

## Constraints

- Code must be compatible with Python 3.10 and above
- Use `pyproject.toml` for configuration

## Running and Debugging

- Execute all code within a virtualenv environment. Create it if missing.

## Imports

- **STRICTLY** use relative imports

## Data Models

- Use pydantic for data models and request/response schemas
- All data model classes must use `kw_only=True` (Python 3.10+ syntax)
- Set empty defaults (empty string, zero, etc.) for all fields. Use `None` only where explicitly required.

## Logging

Preferred standard:
- logging

```python
import logging

logger = logging.getLogger(__name__)
```

Allowed:
- structlog for structured logging
- loguru only in standalone CLI utilities or scripts

Logging Principles:
- Use structured logging where possible

Operational logs **MUST**:
- include contextual metadata
- be machine-readable
- support filtering and aggregation

Prefer:

```python
logger.info(
    "user created",
    extra={
        "user_id": user.id,
        "email": user.email,
    },
)
```

Formatting:
- lowercase messages
- concise operational wording
- stable log event names

Log levels MUST reflect operational importance and required reaction.

### DEBUG

DEBUG is used for diagnostic information useful during development or incident investigation.

Use DEBUG for:
- intermediate state
- request/response details
- branch decisions
- retries
- external payload previews
- performance diagnostics

### INFO

INFO is used for important normal business operations.

INFO logs SHOULD describe:
- lifecycle events
- significant state transitions
- externally observable operations

### WARNING

WARNING indicates abnormal but recoverable situations.
The operation continues, but attention MAY be required.

Use WARNING when:
- fallback logic is activated
- retryable failures occur
- degraded behavior is detected
- unexpected input is received

Use ERROR when:
- a request fails
- data cannot be persisted
- external dependency prevents operation completion
- invariant violation affects functionality

Use CRITICAL only when:
- the process cannot continue
- data corruption is possible
- critical infrastructure is unavailable
- operator intervention is required immediately

## Code Formatting

- Inside function and method bodies, logical blocks are separated by **one blank line**:
  - Variable initialization is separated from conditional constructs and loops
  - Loops and conditions must be separated by a blank line
  - Data preparation is separated from its processing
  - Processing is separated from returning the result
  - Add a blank line for visually dense or hard-to-read blocks

## Docstrings

- All public functions, methods, and classes **MUST** have docstrings
- Docstring format — **Google style**:
  ```python
  def function_name(param1: str, param2: int = 0) -> bool:
      """Brief description of the function.

      Detailed description when necessary.

      Args:
          param1: Description of the first parameter.
          param2: Description of the second parameter.

      Returns:
          Description of the return value.

      Raises:
          ValueError: Condition that triggers this exception.
      """
  ```
- The brief description (first line) is required, starts with a capital letter, and ends with a period
- Include `Args`, `Returns`, `Raises` sections only where applicable

## Dependencies

All third-party libraries **MUST** be added to `pyproject.toml` with a minimum version specified if it is critical for backward compatibility.

---

# Testing

## Constraints

- Test code must be compatible with Python 3.10 and above

## Tools

- pytest — running tests
- ruff — linting and formatting test code
- pytest-cov — coverage

## Running Tests and Linter

- Execute all tests within a virtualenv environment. Create it if missing.

## Test Structure

- Tests mirror the source code structure **directly**, without an intermediate root package directory:
  - `<src>/module/file.py` → `tests/module/test_file.py`
  - The `tests/` directory contains subdirectories corresponding to nested packages of the root package
  - Tests for root package modules (`__main__.py`, `__init__.py`) are placed directly in `tests/` (e.g., `tests/test_main.py`)
- Each new test directory contains an `__init__.py`
- Fixtures are located in `tests/<package>/conftest.py` (local) or `tests/conftest.py` (global, only for shared fixtures)
- Integration tests covering multiple packages are placed directly in `tests/` (e.g., `tests/test_integration.py`, `tests/test_integration_<scenario>.py`)

## Naming

- Files: `test_<module>.py`
- Functions: `test_<what>_<scenario>` (e.g., `test_complexity_with_empty_input`)
- Grouping: `class Test<Component>:`

## Test Types

- **Unit** — every public function/method/class, main scenario and typical data
- **Edge cases**:
  - Empty inputs: `None`, `""`, `[]`, `{}`
  - Boundary values: `0`, negative, very large
  - Invalid types
  - Expected exceptions via `pytest.raises`
- **Integration** — only for interaction between modules/packages

## REST API Testing

- Test endpoints by calling the handler function directly via Python call.

## CLI Testing

- Test CLI commands by calling the command handler function directly via Python call.

## Boundary Tests

- For thresholds, ranges, state transitions — use `@pytest.mark.parametrize` with a table of values including each boundary

## Mocks

- Pure logic — no mocks
- File I/O — use the `tmp_path` fixture exclusively
- Subprocesses — `mock.patch` the subprocess call
- External dependencies — `mock.patch` at the import point

## Miscellaneous

- Use self-documenting test names. Keep comments minimal.
- Skip integration tests with unavailable external dependencies via `pytest.mark.skipif`

# Dependencies

All libraries for testings **MUST** be added to `pyproject.toml`
Testing dependencies must be specified separately in the `[project.optional-dependencies]` section under the `test` key.

---

# Validation Commands

All commands must run in a virtualenv environment. Python 3.10+ compatibility required.

| Purpose                  | Command                                   |
|--------------------------|-------------------------------------------|
| Run all tests            | `pytest tests/ -x`                        |
| Run a specific test      | `pytest tests/test_<name>.py -v`          |
| Lint                     | `ruff check <src>/`                       |
| Facade check             | `python -c "from package import Entity"`  |
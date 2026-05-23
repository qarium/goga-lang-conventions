# General Conventions

## Development

### Constraints

- Code must be compatible with Python 3.10 and above

### Running and Debugging

- Code must be executed within a virtualenv environment
- If a virtualenv environment is not available, it must be created

### Imports

- **STRICTLY** use relative imports

### Data Models

- The pydantic library is used to describe data structures and request/response models
- All data model classes must use kw_only=True
- Structures are created with empty defaults (empty string, zero, etc.) unless None is explicitly specified

### Code Formatting

- Inside function and method bodies, logical blocks are separated by **one blank line**:
  - Variable initialization is separated from conditional constructs and loops
  - Loops and conditions must be separated by a blank line
  - Data preparation is separated from its processing
  - Processing is separated from returning the result
  - If a block of code becomes visually dense or hard to read — add a blank line for readability

### Docstrings

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
- The `Args`, `Returns`, `Raises` sections — only if the function accepts arguments, returns a value, or raises exceptions

## Testing

### Constraints

- Test code must be compatible with Python 3.10 and above

### Tools

- pytest — running tests
- ruff — linting and formatting test code
- pytest-cov — coverage

### Running Tests and Linter

- Execution must be done within a virtualenv environment
- If a virtualenv environment is not available, it must be created

### Test Structure

- Tests mirror the source code structure **directly**, without an intermediate root package directory:
  - `<src>/module/file.py` → `tests/module/test_file.py`
  - The `tests/` directory contains subdirectories corresponding to nested packages of the root package
  - Tests for root package modules (`__main__.py`, `__init__.py`) are placed directly in `tests/` (e.g., `tests/test_main.py`)
- Each new test directory contains an `__init__.py`
- Fixtures are located in `tests/<package>/conftest.py` (local) or `tests/conftest.py` (global, only for shared fixtures)
- Integration tests covering multiple packages are placed directly in `tests/` (e.g., `tests/test_integration.py`, `tests/test_integration_<scenario>.py`)

### Naming

- Files: `test_<module>.py`
- Functions: `test_<what>_<scenario>` (e.g., `test_complexity_with_empty_input`)
- Grouping: `class Test<Component>:`

### Test Types

- **Unit** — every public function/method/class, main scenario and typical data
- **Edge cases** — empty inputs (`None`, `""`, `[]`, `{}`), boundary values (`0`, negative, very large), invalid types, expected exceptions via `pytest.raises`
- **Integration** — only for interaction between modules/packages

### REST API Testing

- Endpoint testing is performed by calling the handler function directly (not via HTTP client).

### CLI Testing

- CLI command testing is performed by calling the command handler function directly, without using external execution interfaces.

### Boundary Tests

- For thresholds, ranges, state transitions — use `@pytest.mark.parametrize` with a table of values including each boundary

### Mocks

- Pure logic — no mocks
- File I/O — `tmp_path` fixture, **never mock `builtins.open`**
- Subprocesses — `mock.patch` the subprocess call
- External dependencies — `mock.patch` at the import point

### Miscellaneous

- Minimal comments — test names must be self-documenting
- Integration tests with external dependencies use `pytest.mark.skipif` when tools are unavailable

## Validation Commands

| Purpose                  | Command                                   |
|--------------------------|-------------------------------------------|
| Run all tests            | `pytest tests/ -x`                        |
| Run a specific test      | `pytest tests/test_<name>.py -v`          |
| Lint                     | `ruff check <src>/`                       |
| Facade check             | `python -c "from package import Entity"`  |

## Dependencies

All third-party libraries **MUST** be added to `pyproject.toml` with a minimum version specified if it is critical for backward compatibility.
Testing dependencies must be specified separately in the `[project.optional-dependencies]` section under the `test` key.
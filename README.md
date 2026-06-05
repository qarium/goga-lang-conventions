# Goga Language Conventions

Coding conventions for goga ecosystem projects. Used as context for AI assistants and as a developer reference.

## How to Use

**As a URL in CODEMANIFEST**:

```yaml
Usages:
  conventions: https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/<lang>/<convention>.md

Annotations: |
  Use `conditions` to write code rules and tests.
```

**As a local usage file in CODEMANIFEST**:

1. Download the file

```bash
curl -o .goga/usages/conventions.md \
  https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/<lang>/<convention>.md
```

2. Include as a local md document

```yaml
Usages:
  conventions: .goga/usages/conventions.md

Annotations: |
  Use `conditions` to write code rules and tests.
```

## Available Conventions

### JavaScript — Project

|           |                                                                                                              |
|-----------|--------------------------------------------------------------------------------------------------------------|
| **File**  | `javascript/project.md`                                                                                      |
| **URL**   | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/javascript/project.md) |
| **Stack** | ES2024/ES2025, Node.js 22+, ESM, Prettier, Jest, ESLint                                                      |

Rules for JavaScript projects: ESM-only modules, async/await, errors, dependency injection, null safety, logging, formatting, JSDoc, naming, dependencies, tests, validation commands.

### Go — Project

|           |                                                                                                          |
|-----------|----------------------------------------------------------------------------------------------------------|
| **File**  | `golang/project.md`                                                                                      |
| **URL**   | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/golang/project.md) |
| **Stack** | Go 1.23+, go.mod, goimports, slog, testing, testify, golangci-lint, govulncheck                          |

Rules for Go projects: imports, constructor dependency injection, context usage, errors, logging, goroutines, composition, formatting, documentation, dependencies, tests, validation commands.

### Kotlin — Project

|           |                                                                                                          |
|-----------|----------------------------------------------------------------------------------------------------------|
| **File**  | `kotlin/project.md`                                                                                      |
| **URL**   | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/kotlin/project.md) |
| **Stack** | Kotlin 2.0.21+, Gradle Kotlin DSL, version catalog, coroutines, ktlint, JUnit 5, Kotest, MockK           |

Rules for Kotlin projects: imports, constructor dependency injection, coroutines, typed domain errors, nullability, data models, CODEMANIFEST contract signatures, logging, composition, formatting, KDoc, dependencies, tests, validation commands.

### Python — Project

|          |                                                                                                          |
|----------|----------------------------------------------------------------------------------------------------------|
| **File** | `python/project.md`                                                                                      |
| **URL**  | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/python/project.md) |
| **Stack** | Python 3.10+, pyproject.toml, pydantic, ruff, pytest                                                    |

Rules for Python projects: virtualenv usage, imports, pydantic data models, logging, formatting, Google-style docstrings, dependencies, tests, validation commands.

### Swift — Project

|           |                                                                                                         |
|-----------|---------------------------------------------------------------------------------------------------------|
| **File**  | `swift/project.md`                                                                                      |
| **URL**   | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/swift/project.md) |
| **Stack** | Swift 6.0.3/6.1.3/6.2.4+, Swift Package Manager, strict concurrency, swift-format, SwiftLint, Testing  |

Rules for Swift projects: imports, access control, value/reference types, initializer dependency injection, strict concurrency, typed throws, logging, formatting, documentation, dependencies, tests, validation commands.

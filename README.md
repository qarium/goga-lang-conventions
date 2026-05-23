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

### Python — Project

|          |                                                                                                          |
|----------|----------------------------------------------------------------------------------------------------------|
| **File** | `python/project.md`                                                                                      |
| **URL**  | [raw](https://raw.githubusercontent.com/qarium/goga-lang-conventions/refs/heads/0.0.x/python/project.md) |
| **Stack** | Python 3.10+, pyproject.toml, pydantic, ruff, pytest                                                    |

Rules for Python projects: code and test structure, imports, data models, formatting, docstrings, dependencies.
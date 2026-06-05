# General Conventions

Mandatory rules for all Swift code in this project. Strict deterministic specification.

## Engineering Principles

Code in this repository MUST prioritize:
- readable, explicit code
- predictable, straightforward control flow
- stable abstractions
- operational stability and maintainability

---

# Development

## Constraints

- Compatible with Swift 6.0.3, 6.1.3, 6.2.4 and above
- Use `Package.swift` for dependency management via Swift Package Manager
- Strict concurrency mode is enabled by default — all code MUST be concurrency-safe

## Imports

RULES:
1. Use explicit module imports — wildcard imports are **STRICTLY** forbidden
2. Group imports in order: standard library, third-party modules, project modules

```swift
// Valid
import Foundation
import Logging
import MyModuleModels

// Forbidden
import MyModule.*
```

All subsequent import patterns MUST follow explicit named imports only.

## Access Control

Only `public` declarations form the module facade (contract). `internal` (default) and `private` do not enter the contract.

RULES:
1. Use `public` for all facade declarations
2. Use `internal` for implementation details shared within the module
3. Use `private` for implementation details scoped to the declaration
4. Prefer `let` over `var` — use `var` only when mutation is required
5. Use `private(set)` for public reads with internal mutation

Every `public` declaration is a contract. Changes to public API require explicit review.

## Value and Reference Types

RULES:
1. Use `struct` (value type) by default
2. Use `class` (reference type) only when identity semantics or shared mutable state is required
3. Use `actor` when shared mutable state requires isolation

```swift
// Value type — default choice
public struct User {
    public let id: UUID
    public var name: String
}

// Reference type — only when identity or shared mutation is needed
public final class SessionManager {
    public private(set) var isActive: Bool = false
}

// Actor — shared mutable state with isolation
public actor TokenStore {
    private var tokens: [String: String] = [:]
}
```

Mark types crossing concurrency boundaries as `Sendable`.

## Dependency Injection

Use initializer-based dependency injection. Dependencies MUST be passed as explicit initializer parameters.

```swift
public final class Service {
    private let repository: Repository
    private let logger: Logger

    public init(
        repository: Repository,
        logger: Logger
    ) {
        self.repository = repository
        self.logger = logger
    }
}
```

Allowed DI patterns:
- initializer injection with explicit parameters
- protocol-based dependencies
- manual wiring

Use `Logging` (swift-log) instead of:
- service locators
- runtime DI containers
- singletons as implicit dependencies

## Concurrency

Swift 6 enforces strict concurrency. All code MUST satisfy Sendable and isolation requirements.

RULES:
1. Use `async/await` for asynchronous operations
2. Use `actor` for shared mutable state requiring isolation
3. Conform to `Sendable` for values crossing concurrency boundaries
4. Use structured concurrency (`TaskGroup`, `async let`) — avoid unstructured `Task` without explicit lifecycle management

```swift
public struct ServiceConfig: Sendable {
    public let baseURL: String
    public let timeout: TimeInterval
}

public func fetchData(endpoint: String) async throws -> Data {
    ...
}
```

Every concurrent operation MUST have:
- a deterministic completion path
- proper cancellation handling
- bounded execution scope

## Error Handling

Use Swift's error handling. Errors MUST:
- conform to `Error` protocol
- be thrown explicitly with `throws`
- preserve error context

```swift
public enum ServiceError: Error {
    case networkUnavailable
    case invalidResponse(String)
    case unauthorized
}
```

Use typed throws when the error set is known and stable:

```swift
public func fetchUser(id: UUID) async throws(ServiceError) -> User {
    ...
}
```

Error classification MUST use pattern matching on error types. Do not compare error descriptions.

Every thrown error MUST be handled by the caller. Propagate errors with `try` or handle with `do/catch`.

## Logging

RULES:
1. Use `Logging` (swift-log) as the default logging library
2. Use `os.Logger` only on Apple platforms when system integration is required

```swift
import Logging

let logger = Logger(label: "com.example.service")
```

Operational logs **MUST**:
- include contextual metadata
- be machine-readable
- support filtering and aggregation

```swift
logger.info("user created", metadata: [
    "user_id": "\(user.id)",
])
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

### ERROR

ERROR is used when an operation cannot be completed.

Use ERROR when:
- a request fails
- data cannot be persisted
- external dependency prevents operation completion
- invariant violation affects functionality

### CRITICAL

CRITICAL is reserved for conditions requiring immediate operator intervention.

Use CRITICAL only when:
- the process cannot continue execution
- data corruption is detected or imminent
- core infrastructure dependencies are unreachable
- manual operator intervention is required within minutes

### Log Content Restrictions

Log messages MUST contain only non-sensitive operational data. Exclude secrets, credentials, tokens, and personal sensitive data from all log output.

## Code Formatting

RULES:
1. All code MUST be formatted with `swift-format`
2. Use `SwiftLint` for additional lint rules

Inside function and method bodies, logical blocks are separated by **one blank line**:
- Variable initialization is separated from conditional constructs and loops
- Loops and conditions are separated by a blank line
- Data preparation is separated from its processing
- Processing is separated from returning the result

Style Rules:
- short functions
- early returns via `guard` statements
- explicit variable naming
- minimal nesting

Write functions as flat sequences of steps. Extract logic into named helpers instead of nesting conditionals beyond two levels.

## Documentation

All public declarations MUST have doc comments.

```swift
/// Creates a new user in the system.
/// - Parameter request: The user creation request.
/// - Returns: The created user.
/// - Throws: ``ServiceError`` if creation fails.
public func createUser(_ request: CreateUserRequest) async throws -> User
```

Comments SHOULD explain intent, invariants, and non-obvious decisions.
Comments SHOULD NOT restate code behavior.

## Dependencies

All dependencies **MUST** be declared in `Package.swift`. Use `.upToNextMinor` or `.upToNextMajor` for version bounds.

---

# Testing

## Constraints

- Test code must be compatible with Swift 6.0.3, 6.1.3, 6.2.4 and above

## Tools

- `Testing` (Swift Testing framework) — primary test framework
- `XCTest` — legacy only, migration to Swift Testing is encouraged
- SwiftLint — linting test code

Prefer Swift Testing (`@Test`, `@Suite`) for all new tests.

## Running Tests

- Run all commands from the package root directory

## Test Structure

Tests mirror the source code structure **directly**:
- `Sources/Module/Service.swift` → `Tests/ModuleTests/ServiceTests.swift`
- Test targets are declared in `Package.swift`
- Shared test helpers are placed in `Tests/Common/`

## Naming

- Files: `<Module>Tests.swift` or `<Type>Tests.swift`
- Functions: descriptive names via `@Test("description")`
- Grouping: `@Suite("Component")`

```swift
@Suite("UserService")
struct UserServiceTests {
    @Test("creates user with valid input")
    func createUserSuccess() async throws {
        ...
    }

    @Test("rejects invalid email")
    func createUserInvalidEmail() async throws {
        ...
    }
}
```

## Test Types

- **Unit** — every public function/method/type, main scenario and typical data
- **Edge cases**:
  - Empty inputs: `nil`, `""`, empty arrays, empty dictionaries
  - Boundary values: `0`, negative, very large
  - Invalid types
  - Expected errors via `#require(throws:)`
- **Integration** — only for interaction between modules or external services

## Boundary Tests

For thresholds, ranges, state transitions — use parameterized tests with a table of values including each boundary.

```swift
@Suite("Normalization")
struct NormalizationTests {
    @Test(arguments: [
        (" test ", "test"),
        ("hello", "hello"),
        ("", ""),
    ])
    func normalizesInput(input: String, expected: String) {
        let result = normalize(input)
        #expect(result == expected)
    }
}
```

## Mocking

- Pure logic — no mocks
- External dependencies — protocol-based test doubles
- Mock at the protocol level, not at the implementation level

Generate mocks only for external dependency protocols. Keep business logic mock-free.

Use hand-written protocol conformances for test doubles.

## Concurrency Testing

Concurrency-sensitive code SHOULD include:
- race condition tests
- cancellation tests
- isolation boundary tests

## Miscellaneous

- Use self-documenting test names via `@Test("description")`. Keep comments minimal.
- Skip integration tests with unavailable external dependencies via `@Test(.enabled(if:))`

## Dependencies

All test dependencies **MUST** be declared in `Package.swift` under the test target.

---

# Validation Commands

All validation commands MUST pass in CI. Swift 6.0.3+ compatibility required.

| Purpose               | Command                              |
|-----------------------|--------------------------------------|
| Format code           | `swift-format format -i -r Sources/` |
| Run tests             | `swift test`                         |
| Run tests in parallel | `swift test --parallel`              |
| Run a specific test   | `swift test --filter ModuleTests`    |
| Lint                  | `swiftlint lint Sources/`            |
| Build                 | `swift build`                        |
| Build for release     | `swift build -c release`             |

# General Conventions

Mandatory rules for all Kotlin code in this project. Strict deterministic specification.

## Engineering Principles

Code in this repository MUST prioritize:
- readable, explicit code
- predictable, straightforward control flow
- stable abstractions
- operational stability and maintainability

---

# Development

## Constraints

- Compatible with Kotlin 2.0.21 and above only
- Use Gradle with Kotlin DSL (`build.gradle.kts`) for build configuration
- Use Gradle version catalog (`gradle/libs.versions.toml`) for dependency version management

## Imports

RULES:
1. Use import aliases only to resolve naming conflicts or when the imported name is misleading
2. Use only named imports — star imports (`*`) are **STRICTLY** forbidden in production code
3. Remove all unused imports

Configure `.editorconfig` with `ij_kotlin_imports_layout` if custom import grouping is required.

```kotlin
// Valid
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

// Forbidden
import kotlinx.coroutines.flow.*
```

All subsequent import patterns MUST follow named imports only.

## Dependency Injection

Use constructor-based dependency injection. Dependencies MUST be passed as explicit constructor parameters.

```kotlin
class Service(
    private val repository: Repository,
    private val logger: Logger,
) {
    // ...
}
```

Allowed DI patterns:
- constructor injection with explicit parameters
- interface-based dependencies
- manual wiring

Use constructor parameter defaults for optional dependencies.

Do NOT use:
- service locators
- runtime DI containers (Dagger, Koin, Hilt) unless mandated by the platform
- reflection-heavy frameworks
- implicit dependencies

## Coroutines and Concurrency

Every coroutine MUST:
- have a lifecycle owner (a `CoroutineScope`)
- terminate predictably
- support cancellation

Structured concurrency MUST be used. Every coroutine launched within a scope MUST complete before the scope completes.

```kotlin
class Worker(
    private val dispatcher: CoroutineDispatcher,
    private val repository: Repository,
) {
    suspend fun process(requests: List<Request>): List<ProcessedItem> =
        coroutineScope {
            requests.map { request ->
                async(dispatcher) {
                    repository.handle(request)
                }
            }.awaitAll()
        }
}
```

Concurrency patterns MUST use:
- `coroutineScope` / `supervisorScope` for structured concurrency
- bounded concurrency via `Semaphore` or channel-based worker pools
- `Flow` for cold streams of values
- `SharedFlow` / `StateFlow` for hot observable state
- explicit cancellation propagation

Dispatchers MUST be injected as `CoroutineDispatcher`, not hardcoded. Use `Dispatchers.Default` as the fallback. Keep `Dispatchers.IO` for blocking I/O operations only.

Shared mutable state in concurrent contexts MUST use thread-safe constructs:
- `AtomicInteger`, `AtomicReference` for single variables
- `Mutex` for complex critical sections
- immutable data transfers between coroutines

## Error Handling

Use sealed interfaces for domain error hierarchies. Errors MUST:
- be returned explicitly as part of the type system
- preserve the original error cause
- include contextual information

Define a project-level `Outcome<out E, out T>` type for typed error returns. Kotlin stdlib `Result<T>` wraps only `Throwable` failures and is NOT suitable for domain errors.

```kotlin
sealed interface Outcome<out E, out T> {
    data class Success<out T>(val value: T) : Outcome<Nothing, T>
    data class Failure<out E>(val error: E) : Outcome<E, Nothing>
}

sealed interface UserError {
    data class NotFound(val userId: String) : UserError
    data class ValidationFailed(val reason: String) : UserError
    data class RepositoryFailure(val cause: Throwable) : UserError
}

fun findUser(userId: String): Outcome<UserError, User> {
    val user = runCatching { repository.findById(userId) }
        .getOrElse { e ->
            return Outcome.Failure(UserError.RepositoryFailure(e))
        }

    return user?.let { Outcome.Success(it) }
        ?: Outcome.Failure(UserError.NotFound(userId))
}
```

Use `Outcome<E, T>` for recoverable domain errors in function returns. Use exceptions only for truly exceptional, unrecoverable conditions.

Error classification MUST use `when` exhaustiveness over sealed types. Every sealed subtype MUST be handled.

Every error returned from a function MUST be handled by the caller. Propagate errors up the call chain with cause preservation.

## Nullability

Nullability MUST be explicit in the type system.

RULES:
1. Use `T?` for nullable types, `T` for non-null types
2. Avoid the `!!` operator — use safe alternatives
3. Use `?.let` for null-safe transformations
4. Use Elvis operator `?:` for default values
5. Use `requireNotNull` with a message for precondition checks

```kotlin
// Valid
val name = user?.name ?: "unknown"
val id = requireNotNull(request.id) { "request.id must not be null" }

// Forbidden
val name = user!!.name
```

The `!!` operator is acceptable ONLY in test code where nullability is controlled by the test setup.

## Data Models

RULES:
1. Use `data class` for all data models and request/response schemas
2. Use `val` for all properties — avoid `var` unless mutation is explicitly required by the contract
3. Default to immutable collections (`List`, `Map`, `Set`) over mutable ones (`MutableList`, `MutableMap`, `MutableSet`)
4. Use `value class` for type-safe wrappers of primitive types

```kotlin
data class User(
    val id: String,
    val email: String,
    val name: String = "",
)
```

### CODEMANIFEST Contract Signatures

Types allowed in CODEMANIFEST entity signatures (contract facade):
- `String`, `Int`, `Double`, `Boolean`
- `List<T>`, `Map<String, T>`
- `T?` for nullable
- Data classes and sealed interfaces defined in the contract

Forbidden in CODEMANIFEST entity signatures:
- `Any`
- `vararg`
- Untyped collections (`List<*>`, `Map<*, *>`)

These restrictions apply only to CODEMANIFEST DSL signatures for cross-language contract portability. General Kotlin code may use any well-typed constructs.

## Logging

RULES:
1. Use structured logging via `mu.KotlinLogging` (kotlin-logging) as the default
2. Use direct SLF4J only for services with measured throughput requirements exceeding kotlin-logging overhead
3. Use injected logger instances — do not create loggers inline in hot paths

```kotlin
private val logger = KotlinLogging.logger {}
```

Operational logs **MUST**:
- include contextual metadata
- be machine-readable
- support filtering and aggregation

```kotlin
MDC.put("user_id", user.id)
logger.info { "user created" }
MDC.remove("user_id")
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

### WARN

WARN indicates abnormal but recoverable situations.
The operation continues, but attention MAY be required.

Use WARN when:
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

### Log Content Restrictions

Log messages MUST contain only non-sensitive operational data. Exclude secrets, credentials, tokens, and personal sensitive data from all log output.

## Composition

Prefer interface delegation and composition over class inheritance.

RULES:
1. Use `interface` to define contracts
2. Use delegation (`by`) for composing behavior
3. Use `sealed interface` for closed type hierarchies
4. Inherit from `abstract class` only when sharing implementation state is essential

```kotlin
// Prefer: interface + delegation
interface Cacheable<T> {
    fun get(key: String): T?
    fun put(key: String, value: T)
}

class UserService(
    private val cache: Cacheable<User>,
    private val repository: UserRepository,
) : Cacheable<User> by cache {
    // ...
}
```

Composition MUST use flat constructor parameters for loosely related components.

## Code Formatting

All code MUST be formatted with `ktlint`.

Inside function and method bodies, logical blocks are separated by **one blank line**:
- Variable initialization is separated from conditional constructs and loops
- Loops and conditions are separated by a blank line
- Data preparation is separated from its processing
- Processing is separated from returning the result

Style Rules:
- short functions
- early returns
- explicit variable naming
- minimal nesting
- use expression bodies for simple single-expression functions

Write functions as flat sequences of steps. Extract logic into named helpers instead of nesting conditionals beyond two levels.

```kotlin
// Prefer
fun statusLabel(status: Status): String = when (status) {
    Status.ACTIVE -> "active"
    Status.SUSPENDED -> "suspended"
    Status.DELETED -> "deleted"
}

// Avoid
fun statusLabel(status: Status): String {
    return when (status) {
        Status.ACTIVE -> {
            "active"
        }
        // ...
    }
}
```

## Documentation

All public functions, methods, classes, and interfaces MUST have KDoc comments.

```kotlin
/**
 * Creates a new user in the system.
 *
 * @param request user creation data
 * @return [Outcome] with created [User] on success or [UserError] on failure
 */
fun createUser(request: CreateUserRequest): Outcome<UserError, User>
```

Comments SHOULD explain intent, invariants, and non-obvious decisions.
Comments SHOULD NOT restate code behavior.

## Dependencies

All dependencies **MUST** be declared in the Gradle version catalog (`gradle/libs.versions.toml`) with a minimum version specified.

---

# Testing

## Constraints

- Test code must be compatible with Kotlin 2.0.21 and above

## Tools

- JUnit 5 — running tests
- Kotest assertions — fluent assertions
- MockK — idiomatic Kotlin mocking
- ktlint — linting and formatting test code

Prefer hand-written fakes over generated mocks.

## Running Tests

- Run all commands from the project root directory

## Test Structure

Tests mirror the source code structure **directly**:
- `src/main/kotlin/com/example/service/UserService.kt` → `src/test/kotlin/com/example/service/UserServiceTest.kt`
- Test files reside in the mirrored test source set
- Shared test helpers are placed in `src/testFixtures/kotlin/`
- Integration tests for external dependencies are placed in `src/integrationTest/kotlin/`

RULES:
1. Each test class MUST test exactly one production class
2. Place test fixtures (test data builders, shared helpers) in `testFixtures`
3. Use `@Nested` inner classes to group related test scenarios

## Naming

- Files: `<Component>Test.kt` (e.g., `UserServiceTest.kt`)
- Functions: `` `test <component> <scenario>` `` (e.g., `` `test createUser with valid request returns user` ``)
- Grouping: `@Nested inner class <Scenario>` (e.g., `@Nested inner class CreateUser`)

## Test Types

- **Unit** — every public function/method/class, main scenario and typical data
- **Edge cases**:
  - Empty inputs: `null`, `""`, empty lists, empty maps
  - Boundary values: `0`, negative, `Int.MAX_VALUE`
  - Null variants for nullable parameters
  - Expected errors via sealed type assertions
- **Integration** — only for interaction between modules or external services

## Mocking

- Pure logic — no mocks
- External dependencies — hand-written fakes or `MockK`
- Mock at the interface level, not at the implementation level

Generate mocks only for external dependency interfaces. Keep business logic mock-free.

```kotlin
class FakeUserRepository : UserRepository {
    private val users = mutableMapOf<String, User>()

    override fun findById(id: String): User? = users[id]

    fun givenUser(user: User) {
        users[user.id] = user
    }
}
```

## Coroutines Testing

Coroutine-sensitive code SHOULD include:
- cancellation tests (`cancelAndJoin`)
- timeout behavior tests
- concurrent access tests

Use `runTest` from `kotlinx-coroutines-test` for testing suspend functions.

```kotlin
@Test
fun `test findById returns user`() = runTest {
    val service = UserService(FakeUserRepository())
    val user = service.findById("user-1")

    assertNotNull(user)
    assertEquals("user-1", user.id)
}
```

## Miscellaneous

- Use self-documenting test names. Keep comments minimal.
- Skip integration tests with unavailable external dependencies via `@EnabledIf` or `@Tag`

## Dependencies

All test dependencies **MUST** be declared in the Gradle version catalog under the test section.

---

# Validation Commands

All validation commands MUST pass in CI. Kotlin 2.0.21+ compatibility required.

| Purpose                  | Command                                      |
|--------------------------|----------------------------------------------|
| Format code              | `./gradlew ktlintFormat`                     |
| Run tests                | `./gradlew test`                             |
| Run integration tests    | `./gradlew integrationTest`                  |
| Lint                     | `./gradlew ktlintCheck`                      |
| Dependency vulnerability | `./gradlew dependencyCheckAnalyze`           |
| Build                    | `./gradlew build`                            |

Lint violations require explicit documented justification for any suppression.

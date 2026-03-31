---
name: code-review
description: Perform a comprehensive Android code review covering clean architecture, SOLID principles, Jetpack Compose best practices, and general code quality
allowed-tools: Read, Glob, Grep
---

Perform a thorough code review on the provided file(s) or changed code. Evaluate every aspect below and report findings grouped by category with severity levels: 🔴 Critical, 🟡 Warning, 🟢 Suggestion.

---

## 1. Clean Architecture

- **Layer separation**: Verify that `data`, `domain`, and `presentation` (or `ui`) layers are clearly separated. Flag any direct dependency from `domain` → `data` or `domain` → `presentation`.
- **Dependency rule**: Dependencies must point inward — `presentation` → `domain` → `data` is allowed; the reverse is not. Flag imports that violate this.
- **Use cases / Interactors**: Business logic should live in use case classes, not in ViewModels, Repositories, or UI code. Flag ViewModels that contain business logic beyond simple state mapping.
- **Repository pattern**: Repositories should expose domain models, not DTOs or entities. Flag any leaking of data-layer models into domain or presentation.
- **Mapper usage**: Verify that mapping between layers (Entity ↔ Domain Model ↔ UI Model) happens at layer boundaries, not deep inside business logic.

## 2. SOLID Principles

- **Single Responsibility (SRP)**: Each class and function should have one reason to change. Flag classes doing more than one job (e.g., a ViewModel that also performs network calls directly).
- **Open/Closed (OCP)**: Prefer extension over modification. Flag switch/when blocks that will need editing every time a new type is added — suggest sealed classes, strategy pattern, or polymorphism.
- **Liskov Substitution (LSP)**: Subtypes must be substitutable for their base types. Flag overridden methods that change expected behavior or violate contracts.
- **Interface Segregation (ISP)**: Interfaces should be small and focused. Flag "fat" interfaces that force implementors to provide no-op stubs.
- **Dependency Inversion (DIP)**: High-level modules should depend on abstractions. Flag concrete class dependencies in constructors — suggest injecting interfaces instead.

## 3. Jetpack Compose — Recomposition & Performance

Apply these rules when the reviewed code contains `@Composable` functions:

- **Stability**: Parameters passed to composables should be stable or immutable. Flag mutable collections (`MutableList`, `ArrayList`) or unstable data classes as parameters. Recommend `@Immutable`, `@Stable`, `kotlinx.collections.immutable`, or the Compose compiler stability config file.
- **Lambda allocations**: Flag lambdas created on every recomposition (e.g., inline lambdas referencing outer-scope mutable state). Recommend `remember { }` wrapping or extracting to method references.
- **remember / derivedStateOf misuse**: Flag expensive computations inside composable bodies that are not wrapped in `remember` or `derivedStateOf`. Also flag unnecessary `remember` on cheap/constant values.
- **Key usage in lists**: `LazyColumn` / `LazyRow` items must use a stable, unique `key`. Flag missing keys or keys derived from list index alone.
- **Side effects**: Verify correct usage of `LaunchedEffect`, `DisposableEffect`, `SideEffect`. Flag side-effect logic placed directly in the composable body without an effect handler. Flag `LaunchedEffect(Unit)` when the effect actually depends on changing state.
- **State hoisting**: State should be hoisted to the appropriate level. Flag composables that both own and render state when they should be stateless. Verify unidirectional data flow (state down, events up).
- **Unnecessary recomposition triggers**: Flag `mutableStateOf` or `MutableStateFlow` reads at too high a scope causing entire subtrees to recompose. Recommend scoping reads as narrowly as possible.
- **Modifier best practices**: The first parameter of every public composable should be `modifier: Modifier = Modifier`. Flag composables missing this. Flag hardcoded sizes/padding that should be passed via modifiers.
- **Image & resource loading**: Flag loading images inside composition without `AsyncImage` / Coil or similar. Flag large bitmaps decoded without downsampling.
- **Heavy work in composition**: Flag any I/O, database, or network call triggered directly inside a composable scope.

## 4. Kotlin Best Practices

- **Null safety**: Flag unnecessary `!!` (bang-bang) operators. Recommend `?.`, `?:`, `let`, or `requireNotNull` with a message.
- **Coroutines & threading**: Flag `GlobalScope` usage — recommend `viewModelScope`, `lifecycleScope`, or injected scopes. Flag blocking calls (`Thread.sleep`, blocking I/O) on `Dispatchers.Main`. Verify proper use of `withContext` for dispatcher switching.
- **Scope functions**: Flag deeply nested `let`/`run`/`apply` chains that reduce readability.
- **Data classes**: Flag classes that should be data classes (used purely for holding data but missing `data` modifier).
- **Sealed classes / interfaces**: When modeling finite state, flag open class hierarchies that should be sealed.
- **Extension functions**: Flag utility methods that should be extension functions for better readability.
- **String templates**: Flag string concatenation with `+` — recommend string templates.

## 5. Android-Specific Concerns

- **Lifecycle awareness**: Flag operations that survive configuration changes incorrectly, or that don't respect lifecycle (e.g., collecting flows in `Activity.onCreate` without `repeatOnLifecycle`).
- **Memory leaks**: Flag holding `Context`, `Activity`, or `View` references in long-lived objects (ViewModels, singletons, companion objects).
- **Resource management**: Flag hardcoded strings, dimensions, or colors that should be in resources. Flag missing content descriptions on images/icons for accessibility.
- **Dependency Injection**: Verify constructor injection is used where possible. Flag `lateinit var` dependencies that should be constructor-injected. Flag service locator anti-patterns.
- **Error handling**: Flag empty catch blocks, swallowed exceptions, or missing error states in UI. Recommend `Result` / sealed class based error handling over try-catch in repositories.
- **Thread safety**: Flag shared mutable state without synchronization. Recommend `StateFlow`, `MutableStateFlow`, or `Mutex` over `LiveData` with manual thread management.

## 6. Code Quality & Maintainability

- **Naming**: Flag unclear or misleading names. Variables, functions, and classes should reveal intent.
- **Function length**: Flag functions longer than ~30 lines — recommend extraction.
- **Class length**: Flag classes longer than ~300 lines — recommend decomposition.
- **Magic numbers/strings**: Flag raw literals that should be named constants.
- **Dead code**: Flag unused imports, unreachable branches, commented-out code blocks, and unused parameters.
- **Duplicated code**: Flag repeated logic that should be extracted into a shared function.
- **Coupling**: Flag tightly coupled classes. Recommend event-based communication or mediator patterns where appropriate.
- **Testability**: Flag code that is hard to test — e.g., static method calls, direct instantiation of dependencies, or system clock access. Recommend injection of interfaces/clocks.

## 7. Security

- **Sensitive data**: Flag logging of tokens, passwords, PII, or API keys. Flag hardcoded secrets.
- **Input validation**: Flag missing validation on user input before sending to APIs or databases.
- **Network security**: Flag plaintext HTTP usage. Verify certificate pinning where applicable.
- **Data storage**: Flag sensitive data stored in SharedPreferences without encryption — recommend EncryptedSharedPreferences or DataStore with encryption.

---

## Output Format

After reviewing, produce a report structured as:

### Summary
- Total issues found (by severity)
- Overall assessment (Approve / Approve with suggestions / Request changes)

### Findings
Group findings by category. For each finding:
- **Severity**: 🔴 / 🟡 / 🟢
- **File**: `path/to/File.kt:lineNumber`
- **Issue**: Clear description of the problem
- **Recommendation**: Concrete fix or improvement with a brief code snippet when helpful

### Positive Highlights
Call out well-written code, good patterns, or strong architectural decisions found during the review.

---
name: unit-test
description: Comprehensive Android unit testing skill enforcing MockK, coroutines-test, Turbine, and best practices for writing reliable, maintainable tests.
---

# Android Unit Test Expert Skill

This skill provides authoritative rules and patterns for writing production-quality unit tests in Android/Kotlin projects. It enforces MockK, kotlin-coroutines-test, Turbine, and modern testing best practices.

## Responsibilities

*   **Test Generation**: Writing unit tests for ViewModels, Use Cases, Repositories, and Mappers.
*   **Mocking**: Setting up mocks, stubs, and verifications using MockK.
*   **Coroutine Testing**: Testing suspend functions and coroutine-based code with virtual time.
*   **Flow Testing**: Testing StateFlow, SharedFlow, and cold Flow emissions with Turbine.
*   **Test Quality**: Ensuring isolation, readability, and determinism in all tests.

## Applicability

Activate this skill when the user asks to:
*   "Write tests for this class/function."
*   "Add unit tests."
*   "Test this ViewModel/UseCase/Repository."
*   "Increase test coverage."
*   "Fix a flaky test."

---

## Critical Rules & Constraints

### 1. Test Framework & Libraries

*   **MockK** is the mocking framework. Do NOT use Mockito or Mockito-Kotlin.
*   **kotlin-coroutines-test** (`runTest`, `TestDispatcher`) for all coroutine testing. Do NOT use `runBlocking`.
*   **Turbine** for all Flow testing. Do NOT manually collect flows with `toList()` or custom collectors.
*   Use **Truth** or **AssertK** style assertions when available. Fall back to JUnit assertions if the project does not include them.

### 2. Test Structure

*   Follow the **Arrange / Act / Assert** pattern in every test.
*   One assertion concept per test. Multiple `assert` calls are fine if they verify the same behavior.
*   No logic in tests — no `if`, `for`, `when`, or `try-catch` blocks.
*   Each test must be independent. No shared mutable state between tests.
*   Use `@BeforeEach` (JUnit 5) or `@Before` (JUnit 4) for common setup, not `init` blocks.

### 3. Test Naming

*   Use backtick syntax for readable names that describe behavior.

```kotlin
@Test
fun `should emit loading then success when data is fetched`() { ... }

@Test
fun `should throw exception when user id is empty`() { ... }
```

### 4. Test Organization

*   Group related tests using `@Nested` inner classes (JUnit 5) or clear naming prefixes.
*   Mirror the source structure: `com.app.feature.MyViewModel` → `com.app.feature.MyViewModelTest`.
*   Place test utilities/fakes in a shared `testFixtures` or `test` source set, not in production code.

---

## MockK Rules

### 5. Mock Creation

*   Use `mockk<T>()` for creating mocks. Use `spyk()` only when you need to call real methods.
*   Use `relaxed = true` sparingly — prefer explicit stubbing so tests fail fast on unexpected calls.

```kotlin
// CORRECT — explicit stubbing
private val repository = mockk<UserRepository>()

init {
    coEvery { repository.getUser(any()) } returns testUser
}

// AVOID — relaxed hides missing stubs
private val repository = mockk<UserRepository>(relaxed = true)
```

### 6. Stubbing & Verification

*   Use `coEvery` / `coVerify` for suspend functions, `every` / `verify` for regular functions.
*   Use `slot<T>()` and `capture()` to assert on arguments passed to mocked functions.
*   Call `confirmVerified()` when you need to ensure no unexpected interactions occurred.

```kotlin
@Test
fun `should call repository with correct parameters`() = runTest {
    val idSlot = slot<String>()
    coEvery { repository.getUser(capture(idSlot)) } returns testUser

    useCase.execute("user-123")

    assertEquals("user-123", idSlot.captured)
    coVerify(exactly = 1) { repository.getUser(any()) }
}
```

### 7. MockK Anti-Patterns

*   **NEVER** use `mockkStatic` or `mockkObject` unless absolutely necessary (e.g., extension functions). Prefer refactoring to inject dependencies.
*   **NEVER** mock data classes — instantiate them with real values.
*   **NEVER** mock the class under test — mock only its dependencies.

---

## Coroutine Testing Rules

### 8. Always Use runTest

*   **ALWAYS** use `runTest` for testing coroutine code. NEVER use `runBlocking`.
*   `runTest` provides virtual time and auto-advances delays.

```kotlin
@Test
fun `should load data successfully`() = runTest {
    coEvery { repository.getData() } returns testData

    viewModel.loadData()
    advanceUntilIdle()

    assertEquals(UiState.Success(testData), viewModel.uiState.value)
}
```

### 9. TestDispatcher Injection

*   **ALWAYS** inject `TestDispatcher` into classes under test. Never rely on real dispatchers.
*   Use `StandardTestDispatcher` for precise control over execution order.
*   Use `UnconfinedTestDispatcher` when you need eager/immediate execution.

```kotlin
@Test
fun `should switch to IO dispatcher`() = runTest {
    val testDispatcher = StandardTestDispatcher(testScheduler)
    val repository = UserRepository(ioDispatcher = testDispatcher)

    repository.fetchUser()
    advanceUntilIdle()

    // assertions
}
```

### 10. Virtual Time

*   Use `advanceUntilIdle()` to process all pending coroutines.
*   Use `advanceTimeBy(millis)` to test delay-based logic (e.g., debounce, retry).
*   **NEVER** use `Thread.sleep()` or real `delay()` in tests.

---

## Turbine (Flow Testing) Rules

### 11. Use Turbine's test Extension

*   **ALWAYS** use `.test { }` to collect and assert Flow emissions.
*   Assert each emission in order with `awaitItem()`.
*   End with `cancelAndIgnoreRemainingEvents()` or `ensureNoMoreEvents()`.

```kotlin
@Test
fun `should emit loading then success`() = runTest {
    viewModel.uiState.test {
        assertEquals(UiState.Loading, awaitItem())
        assertEquals(UiState.Success(testData), awaitItem())
        cancelAndIgnoreRemainingEvents()
    }
}
```

### 12. Turbine Error & Completion Testing

*   Use `awaitError()` to assert error emissions.
*   Use `awaitComplete()` for finite flows that complete.

```kotlin
@Test
fun `should emit error when network fails`() = runTest {
    coEvery { repository.getData() } throws IOException("Network error")

    viewModel.loadData()

    viewModel.errorFlow.test {
        val error = awaitItem()
        assertIs<IOException>(error)
        cancelAndIgnoreRemainingEvents()
    }
}
```

### 13. Turbine Anti-Patterns

*   **NEVER** manually collect flows into a list for assertions. Turbine handles timing and cancellation correctly.
*   **NEVER** ignore emissions — assert every `awaitItem()` or explicitly skip with `skipItems(n)`.
*   Set a timeout for long-running flow tests to avoid hanging: `.test(timeout = 5.seconds) { }`.

---

## Test Isolation & Quality

### 14. No Real I/O

*   **NEVER** perform real network calls, database queries, file I/O, or SharedPreferences access in unit tests.
*   Mock all external dependencies. Use fakes for complex interfaces when mocks become unwieldy.

### 15. Determinism

*   Tests must be deterministic — no randomness, no real clocks, no flakiness.
*   Inject `Clock` or time providers for time-dependent logic.
*   Use `TestDispatcher` for coroutine timing — never rely on real thread scheduling.

### 16. Test Data

*   Define test data as constants or factory functions at the top of the test class or in a shared fixture.
*   Use descriptive names: `testUser`, `emptyResponse`, `networkError` — not `data1`, `obj`.
*   Use real instances of data classes, not mocks.

```kotlin
companion object {
    private val testUser = User(id = "1", name = "John", email = "john@test.com")
    private val emptyUserList = emptyList<User>()
    private val networkError = IOException("Connection refused")
}
```

---

## What to Test

### 17. ViewModel Tests

*   Initial state after construction.
*   State transitions: Loading → Success, Loading → Error.
*   User action handling (e.g., button click triggers correct use case call).
*   Error state mapping and recovery.
*   Edge cases: empty data, null fields, rapid successive calls.

### 18. Use Case Tests

*   Business logic correctness — happy path and edge cases.
*   Error propagation and mapping.
*   Correct orchestration of multiple repository calls.
*   Parameter validation.

### 19. Repository Tests

*   Data source coordination (remote vs local, cache logic).
*   Model mapping from Entity/DTO to Domain.
*   Error handling and fallback behavior.

### 20. Mapper Tests

*   All field mappings including defaults and nulls.
*   Edge cases: empty strings, boundary values, missing optional fields.

---

## What NOT to Test

### 21. Do Not Test

*   Framework internals (Room DAO SQL, Retrofit interface generation, Hilt module wiring).
*   Private methods — test them through the public API.
*   Data class `equals`, `hashCode`, `copy`, `toString` — Kotlin generates these.
*   Trivial getters/setters with no logic.
*   Third-party library behavior — trust it or integration-test it separately.

---

## Output Format

When generating tests, produce:

1. **Complete test class** with all imports, setup, and teardown.
2. **Happy path** test(s).
3. **Error path** test(s).
4. **Edge case** test(s).
5. Brief inline comments only where the test intent is not obvious from the name.

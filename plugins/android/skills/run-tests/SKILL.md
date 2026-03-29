---
name: run-tests
description: Run Android unit tests using Gradle
allowed-tools: Bash(./gradlew *)
---

Run the Android test suite.

Steps:
1. Run: `./gradlew test` for unit tests
2. Optionally run: `./gradlew connectedAndroidTest` for instrumentation tests
3. Summarize results: total tests, passed, failed
4. For any failures, show the test name, file, and line number

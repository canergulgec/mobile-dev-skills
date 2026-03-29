---
name: run-tests
description: Run iOS unit and UI tests with xcodebuild
allowed-tools: Bash(xcodebuild *)
---

Run the iOS test suite.

Steps:
1. Detect the `.xcworkspace` or `.xcodeproj` in the current directory
2. Run: `xcodebuild test -scheme <scheme> -destination 'platform=iOS Simulator,name=iPhone 15'`
3. Summarize results: total tests, passed, failed
4. For any failures, show the test name, file, and line number

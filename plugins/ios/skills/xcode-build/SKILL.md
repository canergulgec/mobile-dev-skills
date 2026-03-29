---
name: xcode-build
description: Build the iOS project using xcodebuild
allowed-tools: Bash(xcodebuild *)
---

Build the iOS project using xcodebuild.

Steps:
1. Detect the `.xcworkspace` or `.xcodeproj` in the current directory
2. Run: `xcodebuild -scheme <scheme> -destination 'generic/platform=iOS' build`
3. Report any errors with file path and line number clearly

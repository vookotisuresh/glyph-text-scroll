# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Glyph is an Android app (package: `com.nothing.glyph`) built with Kotlin and Jetpack Compose that integrates with the Nothing Phone Glyph matrix LED hardware via the `com.nothing:glyph-matrix-sdk:1.0.0` SDK (hosted at `https://maven.nothing.tech`).

## Build Commands

This is a Windows environment. Use `gradlew.bat` (not `./gradlew`).

- **Build:** `gradlew.bat build`
- **Assemble debug APK:** `gradlew.bat assembleDebug`
- **All tests:** `gradlew.bat test`
- **Unit tests only:** `gradlew.bat testDebugUnitTest -PtestSuite=unit`
- **Cucumber tests only:** `gradlew.bat testDebugUnitTest -PtestSuite=cucumber`
- **Single test class:** `gradlew.bat test --tests "com.nothing.glyph.ExampleUnitTest"`
- **Instrumented tests:** `gradlew.bat connectedAndroidTest`
- **Clean:** `gradlew.bat clean`

## Architecture

- **Single module** (`:app`) — no feature or library modules
- **UI:** Jetpack Compose with Material Design 3, no XML layouts
- **Theme:** `GlyphTheme` composable in `ui/theme/` supports dark/light mode and dynamic colors (Android 12+)
- **Entry point:** `MainActivity` (ComponentActivity) with edge-to-edge layout
- **Application class:** `MyApplication` in `GlyphApp.kt` — intended initialization point for the Glyph Matrix SDK
- **Target SDK:** 34, **Min SDK:** 24, **Compile SDK:** 34
- **Java/Kotlin target:** 1.8

## Dependencies

Managed via version catalog at `gradle/libs.versions.toml`. Key dependencies:
- Compose BOM 2024.04.01 (ui, material3, tooling)
- AndroidX Core KTX, Lifecycle, Activity Compose, AppCompat
- Nothing Glyph Matrix SDK 1.0.0

## Permissions

The app declares `BLUETOOTH` and `BLUETOOTH_ADMIN` permissions for Glyph hardware communication.

## Build Configuration

- Gradle 8.7 with Kotlin DSL (`.kts` build files)
- Android Gradle Plugin 8.6.0-alpha07
- ProGuard/minification is disabled (`isMinifyEnabled = false`)
- Non-transitive R classes enabled

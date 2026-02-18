# Glyph Scroll Text

An Android app for the Nothing Phone 3 that scrolls custom text (and emoji) across the 25x25 circular Glyph Matrix LED display on the back of the device. Built with Kotlin, Jetpack Compose, and the Nothing Glyph Matrix SDK.

## Features

- **Custom scrolling text** on the Glyph Matrix LED hardware
- **Live circular preview** in-app that mirrors what the actual matrix displays
- **11 font families** selectable via dropdown (Roboto, Monospace, Serif, Cursive, Casual, Condensed, and more)
- **Emoji support** with luminance-based monochromatic rendering and gamma-corrected contrast enhancement
- **Configurable scroll speed** (1-20 levels, from 500ms to 25ms per frame)
- **Scroll direction** (left or right)
- **Vertical positioning** (Top, Center, Bottom)
- **Adjustable text size** (5-25px) and **text width scaling** (0.5x-2.0x)
- **120 Hz dynamic refresh rate** support
- **Long-press Glyph Button** to hot-reload settings without reopening the app

## Requirements

- Nothing Phone 3 (device ID `Glyph.DEVICE_23112`) with system version 20250801 or later
- Android 14 (API 34) or higher
- Android Studio with Kotlin and Compose support

## Build & Install

This is a Windows environment. Use `gradlew.bat` (not `./gradlew`).

```bash
# Build debug APK
gradlew.bat assembleDebug

# Install on connected device
adb install -r app/build/outputs/apk/debug/app-debug.apk

# Build and install in one step
gradlew.bat assembleDebug && adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## Usage

1. **Open the app** and type your text (or paste emoji) in the text field
2. **Adjust settings** as desired (font, size, speed, direction, position)
3. **Tap "Save Settings"** to persist your configuration
4. **Tap "Open Glyph Toys Manager"** to navigate to the system Glyph Toys screen and add "Scroll Text" to your active toy queue
5. **Short-press the Glyph Button** on the back of the phone to cycle to your Scroll Text toy
6. **Long-press the Glyph Button** to hot-reload settings while the toy is active

## Architecture

```
app/
 src/
  main/
   java/com/nothing/glyph/
    MainActivity.kt          # UI, preview, and shared rendering logic
    ScrollTextToyService.kt   # Glyph Toy service for LED hardware
    GlyphApp.kt              # Application class
    ui/theme/                 # Material 3 theme (dark/light, dynamic colors)
   res/                      # Strings, drawables, icons, themes
   AndroidManifest.xml       # Permissions, activity, service registration
  test/
   java/com/nothing/glyph/
    RenderTextBitmapTest.kt   # 18 unit tests (Robolectric)
    cucumber/                 # Cucumber BDD step definitions
   resources/features/        # Cucumber feature files
  libs/
   glyph-matrix-sdk-1.0.aar  # Nothing Glyph Matrix SDK
```

This is a **single-module** Android app with two main components:

### MainActivity

The Compose-based UI with a live animated preview and all controls. Contains the shared rendering functions used by both the preview and the hardware service:

- **`renderTextBitmap()`** — Renders text to a `Bitmap` using Android's `Canvas` and `Paint` APIs. Handles emoji by converting colored pixels to grayscale via luminance calculation (`0.299R + 0.587G + 0.114B`), applying gamma correction (0.6) for enhanced contrast, and gating dim noise below threshold 20. White text pixels pass through with alpha mapped directly to brightness.

- **`calculateFrameDelay()`** — Maps speed level (1-20) to frame delay in milliseconds using `(525 - speed * 25).coerceIn(25, 500)`.

- **`GlyphMatrixPreview`** — Composable that draws an animated 25x25 LED grid on a Compose `Canvas`, clipped to a circle matching the hardware shape. Reads grayscale pixel values from the rendered bitmap and interpolates between LED-off and LED-on colors using `lerp()`.

### ScrollTextToyService

An Android `Service` that implements the Glyph Toy interface. Bound by the Nothing system app when the user selects this toy:

1. Reads saved settings from `SharedPreferences`
2. Calls `renderTextBitmap()` to create the text bitmap
3. Initializes `GlyphMatrixManager` and registers for `Glyph.DEVICE_23112`
4. Runs a `Handler`-based animation loop that composites the text bitmap onto a 25x25 frame bitmap at the current scroll position, then sends it to the SDK via `setMatrixFrame()`
5. Handles Glyph Button long-press events to reload settings and restart scrolling

## Rendering Pipeline

The text-to-LED pipeline ensures the in-app preview matches the hardware output:

```
User Text Input
       |
       v
renderTextBitmap()  ─── shared by both preview & service
  1. Create Typeface from font family name
  2. Measure text width and height with Paint
  3. Draw text onto a Bitmap via Canvas
  4. Post-process pixels:
     - White text (R=G=B=255): alpha -> grayscale brightness
     - Emoji (colored): luminance -> gamma 0.6 boost -> noise gate -> grayscale
       |
       v
  ┌────────────┐         ┌─────────────────────┐
  │  Preview    │         │  Hardware Service     │
  │  (Compose)  │         │  (ScrollTextToyService)│
  │             │         │                       │
  │ Read blue   │         │ Composite onto 25x25  │
  │ channel for │         │ black frame bitmap    │
  │ brightness  │         │                       │
  │             │         │ Send via SDK:          │
  │ lerp() LED  │         │ GlyphMatrixManager    │
  │ off -> on   │         │ .setMatrixFrame()     │
  └────────────┘         └─────────────────────┘
```

Both paths use the same bitmap, so what you see in the circular preview is what appears on the physical matrix.

## Configuration Options

| Setting | Range | Default | Description |
|---------|-------|---------|-------------|
| Text | Any string | `HELLO` | Text to scroll (supports emoji) |
| Font | 11 families | Monospace | Typeface for rendering |
| Text Size | 5-25 px | 12 | Font size in pixels |
| Text Width | 0.5x-2.0x | 1.0x | Horizontal scale factor |
| Scroll Speed | 1-20 | 5 | Frame delay: 500ms (1) to 25ms (20) |
| Direction | Left / Right | Left | Scroll direction |
| Vertical Position | Top / Center / Bottom | Center | Y offset: 0 / 9 / 18 |

Settings are persisted in `SharedPreferences` under the key `glyph_prefs`.

## Available Fonts

| Key | Display Name |
|-----|-------------|
| `default` | Default (Roboto) |
| `monospace` | Monospace |
| `sans-serif` | Sans Serif |
| `serif` | Serif |
| `cursive` | Cursive |
| `casual` | Casual |
| `sans-serif-light` | Sans Serif Light |
| `sans-serif-condensed` | Condensed |
| `sans-serif-medium` | Sans Serif Medium |
| `sans-serif-black` | Sans Serif Black |
| `sans-serif-thin` | Sans Serif Thin |

## Testing

The project has two test suites that can be run independently or together.

### Run all tests

```bash
gradlew.bat test
```

### Unit tests only (Robolectric)

```bash
gradlew.bat test -PtestSuite=unit
```

18 tests in `RenderTextBitmapTest` covering bitmap dimensions, font families, size/width scaling, edge cases, and deterministic output. Uses Robolectric 4.11.1 with `LEGACY` graphics mode (required on Windows — native runtime DLL not available).

### Cucumber BDD tests only

```bash
gradlew.bat test -PtestSuite=cucumber
```

32 scenarios across 3 feature files:

- **`scroll_speed.feature`** — Frame delay calculation for all 20 speed levels
- **`scroll_behavior.feature`** — Scroll initialization, direction, step advancement, wraparound, full cycle
- **`vertical_position.feature`** — Top/Center/Bottom offset mapping and bounds validation

### Run a single test class

```bash
gradlew.bat test --tests "com.nothing.glyph.RenderTextBitmapTest"
```

## Dependencies

Managed via version catalog at `gradle/libs.versions.toml`:

| Dependency | Version | Purpose |
|-----------|---------|---------|
| Kotlin | 1.9.0 | Language |
| Android Gradle Plugin | 8.6.1 | Build system |
| Compose BOM | 2024.04.01 | UI framework |
| Material 3 | (via BOM) | Design system |
| AndroidX Core KTX | 1.10.1 | Kotlin extensions |
| Glyph Matrix SDK | 1.0 | LED hardware control (local AAR) |
| Robolectric | 4.11.1 | Unit test Android framework |
| Cucumber | 7.15.0 | BDD test framework |

## Permissions

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="com.nothing.ketchum.permission.ENABLE" />
```

## SDK Reference

This project uses the Nothing Glyph Matrix SDK (`glyph-matrix-sdk-1.0.aar`). Key classes:

- **`GlyphMatrixManager`** — Connects to the Glyph hardware service, registers the device, and sends frame data
- **`GlyphMatrixFrame`** — Container for matrix display data with layer support (top/mid/low), built via `Builder` pattern
- **`GlyphMatrixObject`** — Represents a single image/bitmap on the matrix with position, brightness, scale, and rotation properties
- **`GlyphToy`** — Constants for Glyph Button event handling (`EVENT_CHANGE`, `MSG_GLYPH_TOY`, etc.)
- **`Glyph`** — Device identifiers (`DEVICE_23112` for Phone 3)

For full SDK documentation, see the [Glyph Matrix Developer Kit](https://github.com/Nothing-Developer-Programme/GlyphMatrix-Developer-Kit) and the [example project](https://github.com/KenFeng04/GlyphMatrix-Example-Project).

## License

The Glyph Matrix SDK is licensed under the Nothing Technology Limited End User License Agreement. See [LICENSE.md](LICENSE.md) for details. Commercial use requires written permission from Nothing — contact [GDKsupport@nothing.tech](mailto:GDKsupport@nothing.tech).

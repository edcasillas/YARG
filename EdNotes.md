## 📅 2025-04-10 — Summary of Day's Work

### 🎯 Goal:
Port YARG to Android and run it on a Fire tablet, starting from a clean project setup.

### ✅ Achievements:
- Successfully built and installed YARG on a Fire tablet.
- Game boots into the main menu.
- Menu navigation works, and some UI is functional.

### 🛠 Key Fixes & Steps:
- Manually built `YARG.Core` and added the resulting DLL to the Unity project to resolve initial compilation issues.
- Addressed native plugin conflicts for `BassNative`, `STB2CSharp`, and `DiscordGameSDK` by excluding their Windows/Linux/Mac `.dll` and `.so` files from Android builds.
- Integrated Android-compatible versions of `libbass.so`, `libbassmix.so`, `libbassopus.so`, and `libbass_fx.so` (for audio playback).
- Switched scripting backend to IL2CPP to unlock ARM64 support and enable correct ABI targeting.
- Ensured both `armeabi-v7a` and `arm64-v8a` architectures were included in the build.
- Cleaned up package name to avoid installation issues on Fire OS.
- Stubbed out MIDI backend initialization from Keijiro’s `Minis` package by copying it into the project and skipping init logic on Android to avoid `DllNotFoundException: RtMidi.dll`.

### 🧱 Remaining Challenges:
- UI appears squashed — likely due to canvas scaling issues or incorrect aspect ratio handling.
- Localization is broken — addressables may not be loading correctly or language data isn’t bundled.
- `DllNotFoundException` for `sqlite3` — native dependency needs to be disabled or replaced.
- No audio or gameplay yet — needs confirmation after addressables and layout issues are resolved.

### ✅ Current Status:
- YARG runs on Fire tablet and displays the main menu with visible UI elements.
- App is stable (no native crashes), and all critical boot-time exceptions have been resolved.
- MIDI input and database systems are stubbed or inactive for now.

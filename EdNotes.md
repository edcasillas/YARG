# Ed's Notes on YARG Setup and Android Build Exploration

This is a personal log documenting my journey to get YARG running on an Android tablet — specifically a Fire tablet. My goal is to understand the build process, resolve platform compatibility issues, and eventually play YARG on a portable device.

---

## 📅 2025-04-10 — Initial Setup & Unity Build Fixes

- Forked [YARG](https://github.com/YARC-Official/YARG) to [my own repo](https://github.com/edcasillas/YARG).
- Installed required tools:
  - Unity 2021.3.36f1 (LTS)
  - .NET SDK 9.0.203
  - Blender
- Opened the project in Unity and encountered a build error: `dotnet` command not found from within Unity.
- Determined this was due to Unity not inheriting environment variables on macOS.
- Created a `LaunchAgents/environment.plist` file to globally expose the PATH to GUI apps.
- Restarted the system — Unity could still not see `dotnet`.
- Manually built `YARG.Core.dll` via terminal using `dotnet build` and copied it to `Assets/Plugins/YARG.Core`.
- Unity compiled successfully after that.
- Resolved plugin conflict with `discord_game_sdk` by disabling one of the redundant plugins (`.bundle`).
- Fixed a `git push` issue caused by an incorrect remote URL format (`https://ecasillas@github.com/...`), which forced bad authentication. Changing it to `https://github.com/...` solved it.

✅ Unity project now opens with **zero errors** and is ready for Android experimentation.

---

## 📅 2025-04-10 — First Successful Unity Run

- Opened the Unity project and ran it from an empty, unsaved scene.
- Observed that it automatically transitioned to the correct initial scene — confirmed that YARG uses an internal scene bootstrapping mechanism.
- Basic menu navigation works well.
- Unfamiliar with gameplay mechanics, but UI behavior appears solid.
- Proceeding to attempt a build for Android using the current project setup.

---

## 📅 2025-04-10 — Android Build Blocked by Color Space Setting

- Attempted to build YARG for Android but the build buttons were disabled.
- Unity displayed:  
  _“In order to build a player, go to player settings to resolve the incompatibility between color space and the current settings.”_
- Investigated and found the issue was due to **Color Space being set to Linear**, which is often not compatible with lower-end Android devices or with certain graphics APIs like OpenGLES.
- Switched **Color Space from Linear to Gamma** under **Project Settings → Player → Android → Other Settings**.
- Decision:  
  > **"Just trying to get this damn thing to run on a Fire tablet."**  
  Opted for Gamma color space to ensure broad device compatibility and keep things simple.

✅ Build buttons are now enabled. Proceeding to build and test on the device.

---

## 📅 2025-04-10 — First Android Build Attempt (Failed)

- Attempted a full build for Android.
- Addressables built successfully and YARG.Core was compiled automatically by the build process.
- Build failed with:
  - Duplicate native plugin error due to `bassopus.dll` being included twice (x86 and x86_64) in the Android build.
  - Warnings related to shaders and lightmapper (not fatal).
- Resolved by configuring both `bassopus.dll` plugins to only apply to Windows, disabling them for Android.

---

## 📅 2025-04-10 — Investigated `BassNative` Plugin for Android Compatibility

- Discovered that `BassNative` contains native audio libraries for Linux, Mac, and Windows — but **not Android**.
- These are critical for music playback in YARG (likely via the BASS audio engine).
- Android build failed because Unity tried to include incompatible Windows `.dll`s in the APK.
- Temporarily excluded all non-Android native libraries to allow the build to complete.
- Music playback on Android is expected to fail or crash until BASS for Android is properly integrated.
- Un4seen's BASS library **does support Android** (see: https://www.un4seen.com/bass.html#android).

→ Next step: attempt a build without BASS and confirm how the app behaves. Then explore integrating Android-compatible native audio libs.

---

## 📅 2025-04-10 — Integrated BASS Library for Android

- Downloaded official Android BASS library from [Un4seen](https://www.un4seen.com/files/bass24-android.zip).
- Extracted and placed `libbass.so` files in:
  - `Assets/Plugins/BassNative/Android/armeabi-v7a/`
  - `Assets/Plugins/BassNative/Android/arm64-v8a/`
- Configured each `.so` file in Unity:
  - Enabled Android platform
  - Set correct CPU architecture (ARMv7, ARM64)
- Disabled desktop BASS plugin files for Android platform

---

## 📅 2025-04-10 — Organized BASS and BASSopus for Android

- Downloaded and extracted Android versions of `libbass.so`, `libbassmix.so`, `libbassopus.so`, and `libbass_fx.so`.
- Created consistent directory structure:
  - `Assets/Plugins/BassNative/Android/armeabi-v7a/`
  - `Assets/Plugins/BassNative/Android/arm64-v8a/`
- Placed `.so` files in appropriate folders based on architecture.
- Updated plugin import settings in Unity:
  - Set platform to Android
  - Assigned correct CPU architecture (ARMv7, ARM64)
- Maintained same layout style as existing Windows, Mac, and Linux plugins for clarity.

---

## 📅 2025-04-10 — Selected Android Architectures for BASS Plugins

- Decided to include only the following architectures:
  - ✅ `armeabi-v7a` (for older Fire tablets and 32-bit ARM)
  - ✅ `arm64-v8a` (for modern Android devices)
- Skipped:
  - ❌ `x86`
  - ❌ `x86_64` (not used on real Android devices, mostly emulators or Chromebooks)

---

## 📅 2025-04-10 — Consideration: Compiling `STB2CSharp` for Android

- Noticed that `STB2CSharp` is part of `YARG.Core`, suggesting it may be a custom or easily portable C# binding.
- Current Android build excludes the Windows `.dll` versions to avoid conflicts.
- May explore compiling the native STB component (`stb_vorbis.c`) for Android using the NDK or converting the code to a managed C# implementation.
- Leaving the door open for a proper Android-native replacement if needed for `.ogg` decoding.

---

## 📅 2025-04-10 — STB2CSharp Build Conflict

- Android build failed due to a conflict between `STB2CSharp.dll` files for `x86` and `x86_64`.
- These are native Windows bindings for STB Vorbis decoding (used to play `.ogg` files).
- Disabled both `.dll` files for Android in Unity plugin import settings.
- No Android-native version of `STB2CSharp` found in project.
- Assuming audio playback will use BASS/BASSopus on Android instead — to be confirmed during testing.

---

## 📅 2025-04-10 — Discord Game SDK Build Conflict

- Android build failed due to duplicate inclusion of `discord_game_sdk.dll` for both `x86` and `x86_64`.
- These are native Windows libraries for the Discord Game SDK (used for rich presence, etc.).
- Confirmed that the SDK is **not supported on Android**.
- Disabled both `.dll` plugins for Android in Unity's plugin import settings.

---

## 📅 2025-04-10 — 🎉 First Successful Android Build Installed

- Successfully built and installed YARG APK onto the Fire tablet!
- Game launches, but UI displays a bare-bones skeleton menu with multiple "LABEL" entries.
- Suspected causes:
  - Localization text not loaded (missing Addressables or failed init)
  - Incomplete resource loading on Android
  - First-time boot flow not triggering properly
  - Potential silent crash or fallback in BASS or other systems

→ Next step: investigate Addressables loading and language manager behavior on Android.

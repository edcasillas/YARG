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

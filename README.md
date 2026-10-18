# Background Location Engine for Flutter

A single-file, reusable Flutter/Dart engine for keeping GPS location tracking
alive and self-healing when the phone is **locked or in a pocket** — the
situation where most naive `geolocator`/`location` integrations quietly stop
delivering updates.

This package provides a reusable implementation for background location
tracking in Flutter applications, with Dart and native Android/iOS
configuration included. Drop it into any Flutter app that needs reliable
location updates while backgrounded or screen-locked.

## What's included

- **`Background_Location_Engine.dart`** — the engine source code.
- **`Background_Location_Engine_Guide.pdf`** — full integration guide (read
  this first).

## Key features

- **Dual-mode tracking**: a lightweight foreground stream while the app is
  on-screen, and a resilient background stream (Android foreground service +
  persistent notification + WAKE_LOCK) while the app is backgrounded or the
  screen is locked.
- **Self-healing stream**: automatically cancels and restarts tracking after
  a configurable delay if the GPS stream errors out.
- **Stale-fix watchdog**: periodically re-publishes the last known fix with
  an explicit `isStale` flag if no fresh fix has arrived recently, so
  consumers never see a silently frozen state.
- **Clean, callback/stream-based API**: `initialize()`, `startForegroundTracking()`,
  `startBackgroundTracking()`, `stopForegroundTracking()`, `stopBackgroundTracking()`,
  `updates` stream, `errors` stream, `dispose()`.
- **Optional app-lifecycle wiring** (`handleAppLifecycleChange`) that
  automatically switches modes on pause/resume.
- **One-shot helpers**: `getLastKnownPosition()`, `getCurrentPosition()`.

## Requirements

- Flutter 3.x, Dart SDK (per your project's `pubspec.yaml`).
- Packages: `geolocator: ^13.0.4`, `flutter_local_notifications: ^21.0.0`
  (only needed if you keep the optional secondary status notification).
- Android: `minSdkVersion` 21+ recommended (matches `geolocator_android`'s
  own requirements). Native manifest permissions documented in the PDF and
  inline in the `.dart` file.
- iOS: `Info.plist` location + background-mode keys documented in the PDF
  and inline in the `.dart` file.
- A device (or emulator with a mocked GPS route) to test — **background
  behaviour cannot be fully verified in a desktop/simulator environment; see
  Limitations below.**

## Installation and quick start

1. Add the required packages to your `pubspec.yaml` (see Requirements above).
2. Copy `Background_Location_Engine.dart` into your project's `lib/` folder
   (e.g. `lib/services/background_location_engine.dart`).
3. Copy the Android manifest block and iOS `Info.plist` block from the top
   comments of the `.dart` file (and the PDF guide, section "Installation
   and Integration") into your own `AndroidManifest.xml` / `Info.plist`.
4. Request location permission in your app (not included — use
   `permission_handler` or your own flow; see the PDF for the required
   two-step Android 11+ request order).
5. Start tracking:

```dart
final engine = BackgroundLocationEngine();
await engine.initialize();
engine.updates.listen((update) {
  print('${update.position.latitude}, ${update.position.longitude}');
});
await engine.startForegroundTracking();
```

Full walkthrough, lifecycle-driven example, and troubleshooting steps are in
`Background_Location_Engine_Guide.pdf`.

## Supported platforms and verified compatibility

| Platform | Status |
|---|---|
| Android | Core logic verified by static analysis (`flutter analyze`, no issues) against `geolocator` 13.0.4 / `geolocator_android` 4.6.2 APIs. **Not re-verified on a physical locked device by this packaging pass** — see Limitations. |
| iOS | `AppleSettings` usage verified against `geolocator_apple` 2.3.13's public API. This is a documented, spec-correct configuration, not a claim of field-tested iOS behaviour — see Limitations. |
| Web / Desktop | Falls back to basic foreground-only `LocationSettings`. No background/locked-screen behaviour applies. |

## Limitations

- **No forced-kill protection.** Per `geolocator_android`'s own
  documentation, the foreground notification increases process priority —
  it does not prevent aggressive OEM battery managers (MIUI, OneUI, ColorOS,
  etc.) from killing the app outright. Auto-restart only recovers from
  *stream errors*, not from the OS killing the whole process.
- **No custom native Kotlin/Swift source is included or required** — the
  real native work is already implemented inside the `geolocator_android`
  / `geolocator_apple` plugins. You still must add the manifest/`Info.plist`
  configuration yourself; this cannot be done from Dart alone.
- **Real-device, screen-locked, long-duration testing is strongly
  recommended before shipping** on both platforms, following the
  procedures in the PDF guide. The logic has been verified via static
  analysis (`flutter analyze`) only as part of this packaging pass.
- Permission requesting/UX is intentionally **not** included — bring your
  own flow (e.g. `permission_handler`).

## License

Proprietary, commercial-use license. You may use and modify the source code
in your own personal and commercial applications. You may **not** resell or
redistribute the engine/source code itself as a standalone or competing
product. See `LICENSE.txt` for full terms, including warranty disclaimers
and sections flagged for your own legal review before relying on this
commercially.

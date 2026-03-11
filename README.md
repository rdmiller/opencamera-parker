# Open Camera Parker

A fork of [Open Camera](https://opencamera.org.uk/) optimized for the **Motorola One Zoom (parker)** and other SM6150/Snapdragon 675 devices running custom ROMs.

## Why this fork?

The Qualcomm SM6150 ADSP firmware has a critical bug: **24-bit PCM audio capture is silently broken**. The DSP accepts 24-bit format requests without error but returns pure silence. This causes video recordings to have no audio on AOSP/LineageOS-based ROMs, since Android's AudioFlinger prefers the (broken) 24-bit path when it's advertised.

Stock Open Camera works around this partially because it allows audio source selection, but the default Camcorder source still triggers silent audio. This fork fixes the problem at the app level and adds hardware-specific optimizations for the Moto One Zoom's triple camera system.

## Changes from upstream

### 1. Force 16-bit audio capture (SM6150 fix)
- Auto-detects SM6150/trinket hardware
- Overrides audio sources that trigger broken 24-bit PCM (Camcorder, Default, Unprocessed) to Voice Recognition, which negotiates 16-bit
- "Force 16-bit audio" toggle in Video settings, auto-enabled on SM6150
- **This fix works without root** - no Magisk module needed

### 2. Zoom quick-switch buttons (0.6x / 1x / 3x)
- Tap-to-zoom preset buttons for ultra-wide, main, and telephoto lenses
- Appears automatically on multi-camera devices (min zoom < 1x)
- Optimized for the Moto One Zoom's camera system:
  - 16MP ultra-wide (f/2.2, 117-degree FOV)
  - 48MP main (f/1.7, OIS)
  - 8MP telephoto 3x (f/2.4, OIS)
- Uses smooth zoom transitions via `CONTROL_ZOOM_RATIO` on Android 11+

### 3. Hybrid OIS+EIS stabilization
- New "Stabilization mode" preference: EIS only (original) or Hybrid (EIS + OIS)
- Stock Open Camera disables OIS when EIS is enabled to avoid conflicts
- Hybrid mode keeps OIS active alongside EIS - essential for the telephoto lens at 81mm equivalent where OIS is critical
- Only shown when the device supports both OIS and video stabilization

### 4. SM6150 audio source warnings
- Detects SM6150 hardware and warns users when selecting audio sources known to produce silent video
- Updated preference summaries explaining which sources are safe
- Guides users toward working configurations if "Force 16-bit audio" is disabled

## Affected devices

**Confirmed:** Motorola One Zoom (parker)

**Likely affected** (same SoC/audio HAL - reports welcome):
- Motorola One Action (troika)
- Motorola One Vision (kane)
- Other SM6150/SM7150 devices with Cirrus Logic CS47L35 codec

## Building

Requires Android Studio (Panda or later) with:
- Android SDK Platform API 35
- JDK 17 (bundled with Android Studio)

```
git clone <this-repo>
cd opencamera
./gradlew assembleDebug
```

The APK installs alongside the original Open Camera (`net.sourceforge.opencamera.parker` vs `net.sourceforge.opencamera`).

## Root users: Magisk module alternative

If you prefer to fix all camera apps system-wide (including the stock Aperture camera), a Magisk module that removes the broken `AUDIO_FORMAT_PCM_8_24_BIT` profile from `/vendor/etc/audio_policy_configuration.xml` is available. Note that Aperture/CameraX apps may not handle the fallback gracefully - Open Camera (this fork or upstream) is the recommended camera app.

## Upstream

- **Open Camera** by Mark Harman: https://opencamera.org.uk/
- Source: https://sourceforge.net/p/opencamera/code/
- Google Play: https://play.google.com/store/apps/details?id=net.sourceforge.opencamera
- Based on upstream version **1.55** (v93)

## License

GPL v3 or later, same as upstream Open Camera. See [gpl-3.0.txt](gpl-3.0.txt).

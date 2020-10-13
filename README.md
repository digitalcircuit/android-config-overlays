# android-config-overlays

A collection of [Runtime Resource Overlays](https://source.android.com/devices/architecture/rros) to customize Android's behavior.

## Configurations

### `config_allowAllRotations`

This allows the screen to be rotated via the accelerometer in all 4 rotations - portrait, landscape, reverse landscape, and reverse portrait (upside-down).  It also enables lockscreen rotation.

### `config_brightnessThresholdsOfPeakRefreshRate`

This forces 60 Hz at or below a `settings get system screen_brightness` of 3, working around the 90 Hz green tint on the Pixel 4 XL.

*Note: unfortunately, [`DisplayModeDirector isInsideZone()`](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/services/core/java/com/android/server/display/DisplayModeDirector.java;l=1280-1298?q=config_brightnessThresholdsOfPeakRefreshRate ) still makes use of the screen brightness integer.  This means it is not possible to distinguish between certain brightness levels with the same accuracy, resulting in either allowing some green tint in (brightness of `2`) or missing out on some non-green-tint brightness (brightness of `3`).*

## Build

*Many thanks to [Know RRO (Runtime Resource Overlay) in Android AOSP](https://saurabhsharma123k.blogspot.com/2018/02/know-rro-runtime-resource-overlay-in.html ) and [How to Completely Take Control of Ambient EQ on the Google Pixel 4](https://www.xda-developers.com/google-pixel-4-ambient-eq-tweak/ ) for the guidance with these steps.*

1.  Set up the Android SDK and build tools for your Android version (newer SDK versions may be backwards compatible)

2.  Change into the directory of the desired overlay, e.g. `config_allowAllRotations`
```
cd <path to folder containing AndroidManifest.xml>
```

3.  Compile the RRO (Runtime Resource Overlay) into an Android package
```
$ ~/Android/Sdk/build-tools/30.0.2/aapt package -M AndroidManifest.xml -S res/ -I ~/Android/Sdk/platforms/android-30/android.jar -F MyOverlayPackage.apk.unsigned
```

4.  Sign the `.apk` file
    * It's probably a good idea [to make your own custom keystore](https://gist.github.com/henriquemenezes/70feb8fff20a19a65346e48786bedb8f ) instead of relying on the debug keystore
```
~/Android/Sdk/build-tools/30.0.2/apksigner sign --ks ~/.android/mykeys.keystore MyOverlayPackage.apk.unsigned
```

5.  Zip-align the `.apk` file
```
~/Android/Sdk/build-tools/30.0.2/zipalign 4 MyOverlayPackage.apk.unsigned MyOverlayPackage.apk
```

6.  Optional: delete the unsigned `.apk` file
```
rm MyOverlayPackage.apk.unsigned
```

## Usage

Place the compiled `.apk` into the appropriate `system` or `vendor` partition within the overlay folder, e.g. `system/vendor/overlay/MyOverlayPackage.apk`.

### Magisk module

1.  Create [a basic Magisk module](https://topjohnwu.github.io/Magisk/guides.html#magisk-modules )

[`module.prop`](https://topjohnwu.github.io/Magisk/guides.html#moduleprop ):
```
id=your_unique_module_id
name=My Awesome Android Overlay
version=v1.0
versionCode=1
author=Awesome Author Name Here
description=Changes the doodad to be more whizbang
```

2.  Create the overlay folder next to `module.prop`, e.g. `system/vendor/overlay`

3.  Place the compiled `.apk` files into the overlay folder above

4.  Compress the `module.prop` and overlay folder next to each other in a `.zip` file

5.  Install using Magisk Manager

6.  Reboot and test your changes, using [Safe Mode in case your device fails to boot](https://topjohnwu.github.io/Magisk/faq.html#q-i-installed-a-module-and-it-bootlooped-my-device-help )

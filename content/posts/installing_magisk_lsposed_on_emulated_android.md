---
title: "Installing Magisk and LSPosed on emulated Android"
date: 2024-05-28
---

After attempting many different ways of getting Magisk to run on Android emulators (including Waydroid and the like), the only way that worked is using the official emulator from Google, along with the instructions from the following Gist:

[tothi/magisk_ramdisk_patch_avd.sh](https://gist.github.com/tothi/1a206791c8b77d7e42015183c980657e)

# Notes

* `userdata.img` needs to be recreated when changing size via the `emulator -partition-size` flag, so pick its size carefully!  The default size of 6 GB can fill up very quickly.

* I placed `ramdisk-patched.img` in `~/.android/avd/android12-play.avd/` to keep it alongside other files for this AVD.

* You may want to avoid using snapshots while getting your AVD setup, but they can be useful later on.

Here are the command-line flags I use to start the emulator:
```sh
emulator @android12-play \
    -partition-size 16384 \
    -memory 8192 \
    -no-audio \
    -feature -Vulkan \
    -gpu host \
    -ramdisk ~/.android/avd/android12-play.avd/ramdisk-patched.img
```

**Note:** I used to specify the number of emulated CPU cores like so: `-cores 8`.  But for whatever reason, this breaks snapshot functionality.

# Userspace installation

After getting the emulated device up and running with a patched `ramdisk.img`, finish setting up Magisk as follows:

1. Install Magisk by sideloading the APK: `adb install Magisk-v27.0.apk`.  See the [GitHub releases page](https://github.com/topjohnwu/Magisk/releases/).

2. Open up Magisk.  (Note that the app may not show up in the apps list until Android initial setup has completed.)  Follow the prompt requesting for a reboot to perform "additional setup".

3. Copy the LSPosed zygisk ZIP over to the device: `adb push LSPosed-v1.9.2-7024-zygisk-release.zip /sdcard/Download/`.  LSPosed is no longer under development, so download the [last available release](https://github.com/LSPosed/LSPosed/releases/download/v1.9.2/) from GitHub.

4. Open up Magisk and select the Modules tab.  Choose "Install from storage" and select the LSPosed ZIP file.  After installing, you will be prompted to reboot again.

5. After rebooting, you should have a silent notification "LSPosed loaded".  Tap to open the LSPosed interface, and choose whether you want to add an app shortcut.

6. Now you can search for modules in the "Repository" tab and install them!

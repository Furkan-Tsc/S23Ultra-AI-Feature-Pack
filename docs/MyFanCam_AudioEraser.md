# Enabling One UI 9 My FanCam & Audio Eraser on S23 Ultra (One UI 8.5)

This guide enables Samsung One UI 9's **My FanCam** (`AutoReframe` / `SUPPORT_VIDEO_REFRAME`) and **Audio Eraser** features on the **Samsung Galaxy S23 Ultra (`SM-S918B` - One UI 8.5)** without touching `/system`.

> [!NOTE]
> The installer preserves your existing **Now Brief / Now Nudge** modifications in `/system/etc/floating_feature.xml` (`/data/local/tmp/ff_s23.xml` or `/data/adb/ff_s23.xml`) and only merges the required tags cleanly.

---

## Requirements

- **Soft Root** (Using *Root My Galaxy*) or **KernelSU / Magisk**
- **LSPosed / Vector** (with ZygiskNext)
- **CorePatch (4.9)** module enabled (signature verification and version downgrade patches enabled — *Disable signature verification*, *Downgrade*, and *Bypass digest* checks active)
- **Termux**
- **Release archive:** Latest **`S23U_MyFanCam_AudioEraser.zip`** from this repository's **[Releases](https://github.com/Furkan-Tsc/S23Ultra-AI-Feature-Pack/releases)** page

---

## Install With Termux

1. Download latest **`S23U_MyFanCam_AudioEraser.zip`** on your phone and extract it to `/sdcard/Download/S23U_MyFanCam_AudioEraser`.
2. Open **Termux** and run:

```sh
su -mm -c "sh /sdcard/Download/S23U_MyFanCam_AudioEraser/install_on_device.sh"
```

The script automatically:
- Sets `ro.build.official.release = false` via `resetprop` to bypass Samsung's `-3001: INSTALL_FAILED_REJECTED_BY_BUILDTYPE` check
- Installs patched native libraries and Audio Eraser models into `/data/adb/fancam/`, `/data/local/tmp/mca/`, `/data/sbs/` 
- Bind-mounts `floating_feature.xml`, `libveframework.videoeditor.samsung.so`, `libcontextanalyzer_jni.media.samsung.so` over `/system`
- Installs `Gallery_15.9_MyFanCam.apk` (`15.9.00.52`) and `VideoEditor_OneUI9_MyFanCam.apk` (`7.5.01.27` / VEKit `4.0.14`)
- Writes a boot hook at `/data/adb/post-fs-data.d/99_z_fancam_audioeraser.sh` and a standalone `/data/local/tmp/mca/reapply.sh` helper

---

## Control Flags (`floating_feature.xml`)

Gallery, Video Editor, Media Context Analyzer (`MCA`) and Audio Eraser check these tags in `/system/etc/floating_feature.xml`:

| Feature / Purpose | `floating_feature.xml` Tag | Original (S23U One UI 8.5) | Required Value |
| :--- | :--- | :--- | :--- |
| Target Tracking | `SEC_FLOATING_FEATURE_MMFW_SUPPORT_TARGET_TRACKING` | Missing | `TRUE` |
| Media Context Analyzer | `SEC_FLOATING_FEATURE_MMFW_SUPPORT_MEDIA_CONTEXT_ANALYZER` | Missing | `TRUE` |
| MCA Accelerator Core | `SEC_FLOATING_FEATURE_MMFW_CONFIG_MEDIA_CONTEXT_ANALYZER_CORE` | Missing | `NPU` |
| Audio Eraser (Multi-Source Separator) | `SEC_FLOATING_FEATURE_AUDIO_CONFIG_MULTISOURCE_SEPARATOR` | Missing | `{FastScanning_6, SourceSeparator_4, Version_1.3.0}` |

---

## How It Works

To make One UI 9 (Android 17) features work on S23 Ultra (Android 16) without touching `/system`, we fixed 4 big problems:

1. **Stopping old and new files from fighting**
   The phone already has an old Samsung video library (`vekit`). When we install the new app, they fight and crash. So we just renamed the new one to `vekjt`. Now they work together nicely with no crashes, and we also unlocked some hidden features.

2. **A translator between Android 17 and Android 16 (`g.so`, see [`src/g.c`](../src/g.c))**
   When playing or editing videos, the new app talks in Android 17 language. Our phone is Android 16 so it gets confused and crashes. We put a small translator file called `g.so` in the middle. It only listens to the new video editor and changes its commands into Android 16 language. It does not touch anything else on the phone.

3. **Moving Audio Eraser files (`/data/sbs/`)**
   Audio Eraser kept giving errors because it could not find the new AI audio files. Since we do not want to touch the system folder, we put those files into `/data/sbs/` and told the app to look there instead. Problem solved.

4. **Making S25 Ultra AI work on S23 Ultra (`libsnap_tfl.so`, see [`src/libsnap_tfl.c`](../src/libsnap_tfl.c))**
   The original My FanCam AI files only work on the S25 Ultra chip (I used S25U dump). S23 Ultra refuses to run them. So we made a bridge called `libsnap_tfl.so`. It uses the human detection AI that is already inside the S23 Ultra, changes the results into what My FanCam wants, and sends them to the app. It is a small trick that makes everything run smoothly on our phone.
---

## Persistence After Reboot

- `/data/adb/post-fs-data.d/99_z_fancam_audioeraser.sh` automatically runs `reapply.sh --no-zygote` on every boot.
- **Soft Root (*Root My Galaxy*):** Because `/system` is untouched (`mount --bind` lives in RAM), after a full hardware reboot re-obtain soft root and run this single command in Termux:

```sh
su -mm -c "sh /data/local/tmp/mca/reapply.sh"
```

---

## Uninstall / Rollback

- **Termux:** Run `su -mm -c "sh /sdcard/Download/S23U_MyFanCam_AudioEraser/uninstall_on_device.sh"`.
- To also revert Gallery and Video Editor back to stock system versions:
  ```sh
  pm uninstall-system-updates com.sec.android.gallery3d
  pm uninstall-system-updates com.sec.android.app.vepreload
  ```
  (Or go to **Settings → Apps → 3-dots menu → Uninstall updates** for both apps.)

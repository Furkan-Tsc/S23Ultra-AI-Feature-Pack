# S23Ultra-AI-Feature-Pack

![Device](https://img.shields.io/badge/Device-Galaxy%20S23%20Ultra%20(SM--S918B)-blue)
![Base](https://img.shields.io/badge/Base-One%20UI%208.5%20%2F%20Android%2016-brightgreen)
![Knox](https://img.shields.io/badge/Knox-0x0%20Preserved-success)
![Root](https://img.shields.io/badge/Root-Soft%20Root%20%2F%20KernelSU-orange)

Enable One UI 9 & Galaxy AI features on the **Samsung Galaxy S23 Ultra (`SM-S918B`, One UI 8.5 / Android 16)** without modifying `/system` (**Knox `0x0` preserved**): **Now Brief**, **Now Nudge**, **Now Bar**, **Notification Highlights**, **My FanCam** and **Audio Eraser**.

---

## Included Features

| Feature | Port Source / Method | Guide |
| :--- | :--- | :--- |
| **Now Brief & Widget** | `floating_feature.xml` bind-mount + Smart Suggestions `7.3.30.9` | [NowBrief_NowNudge.md](docs/NowBrief_NowNudge.md) |
| **Now Nudge** | `floating_feature.xml` (`NOW_NUDGE_VERSION=2`, `LLM_VERSION=1.70`) | [NowBrief_NowNudge.md](docs/NowBrief_NowNudge.md) |
| **Notification Highlights** | `AI_VERSION=20263`, Bypass Samsung Offline Language Model | [NotiHighlightsAI](https://github.com/Furkan-Tsc/Samsung-NotiHighlights-AI), [NowBrief_NowNudge.md](docs/NowBrief_NowNudge.md) |
| **Now Bar Fix (AOD / Lock Screen)** | Shared-UID Samsung-signed bridge via [GenUIBridge.py](tools/GenUIBridge.py) | [NowBarFix.md](docs/NowBarFix.md) |
| **My FanCam (Auto Reframe)** | One UI 9 Video Editor (`vekjt` isolation) + [`g.c`](src/g.c) ABI shim + [`libsnap_tfl.c`](src/libsnap_tfl.c) SNAP→TFLite bridge | [MyFanCam_AudioEraser.md](docs/MyFanCam_AudioEraser.md) |
| **Audio Eraser (v1.3.0)** | One UI 9 `libsbs.so` + `/data/sbs` TFLite/Sorione models + `AudioEraserFeatureParser` fallback | [MyFanCam_AudioEraser.md](docs/MyFanCam_AudioEraser.md) |

---

## Requirements

- **Soft Root** (Using *Root My Galaxy*) or **KernelSU / Magisk**
- **LSPosed / Vector** (with ZygiskNext)
- **CorePatch (4.9)** module enabled (signature verification and version downgrade patches enabled)
- **Termux**

---

## Quick Start

1. **Now Brief, Now Nudge & Notification Highlights:**
   Follow [docs/NowBrief_NowNudge.md](docs/NowBrief_NowNudge.md), then apply [docs/NowBarFix.md](docs/NowBarFix.md) using [tools/GenUIBridge.py](tools/GenUIBridge.py) so cards appear on the lock-screen Now Bar / AOD.
2. **My FanCam & Audio Eraser (One UI 9 Port):**
   Download latest **`S23U_MyFanCam_AudioEraser.zip`** from **[Releases](https://github.com/Furkan-Tsc/S23Ultra-AI-Feature-Pack/releases)** and follow [docs/MyFanCam_AudioEraser.md](docs/MyFanCam_AudioEraser.md):
   - **Termux:** Extract to `/sdcard/Download/S23U_MyFanCam_AudioEraser` and run:
     ```sh
     su -mm -c "sh /sdcard/Download/S23U_MyFanCam_AudioEraser/install_on_device.sh"
     ```

---

## Persistence After Reboot

Because `/system` stays completely untouched (every modification runs over `mount --bind` in RAM), a full hardware reboot clears active bind-mounts until you re-obtain soft root.

- **If you installed My FanCam & Audio Eraser** (with or without Now Brief), re-apply **all** bind-mounts and reload `SemFloatingFeature` in one command:
  ```sh
  su -mm -c "sh /data/local/tmp/mca/reapply.sh"
  ```
- **If you only installed Now Brief / Now Nudge**, run:
  ```sh
  su -mm -c '
  mount --bind /data/local/tmp/ff_s23.xml /system/etc/floating_feature.xml
  setprop ctl.restart zygote
  '
  ```

*Also you get automatic boot persistence via `/data/adb/post-fs-data.d/99_z_fancam_audioeraser.sh`.)*

---

## Repository Structure

```text
S23Ultra-AI-Feature-Pack/
├── README.md                              # Overview, feature matrix & quick start
├── docs/
│   ├── NowBrief_NowNudge.md              # Now Brief, Now Nudge & Notification Highlights guide
│   ├── NowBarFix.md                       # AOD / Lock Screen Now Bar permission fix
│   └── MyFanCam_AudioEraser.md            # One UI 9 My FanCam & Audio Eraser guide
├── scripts/
│   ├── install_on_device.sh               # On-device / Termux installer
│   └── uninstall_on_device.sh             # On-device / Termux uninstaller
├── tools/
│   └── GenUIBridge.py                     # AODService / SystemUI shared-UID bridge builder
└── src/
    ├── g.c                                # Android 17 -> 16 libgui.so BufferQueue ABI bridge
    └── libsnap_tfl.c                      # SNAP -> TFLite human detector & spatial ReID bridge
```

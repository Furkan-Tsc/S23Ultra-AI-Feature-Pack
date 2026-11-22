# Enabling Now Brief, Now Nudge & Notification Highlights on S23 Ultra

This guide turns on **Now Brief**, **Now Nudge**, **Smart Suggestions Widget**, **Galaxy AI**, and **Notification Highlights** on the Samsung Galaxy S23 Ultra (`SM-S918B` - One UI 8.5). It does not touch `/system`, so **Knox stays `0x0`**.

> [!TIP]
> To see **Now Brief** cards on your lock screen (Now Bar / AOD), also follow **[NowBar Fix](NowBarFix.md)**.
> If you also want One UI 9's **My FanCam** and the new **Audio Eraser**, check **[My FanCam & Audio Eraser](MyFanCam_AudioEraser.md)** (it keeps your `ff_s23.xml` changes safe).

## What You Need

- Soft Root (Root My Galaxy) or KernelSU / Magisk
- LSPosed / Vector (with ZygiskNext)
- CorePatch (4.9) module (turn on signature and downgrade patches)
- Termux
- [NowBar Fix](NowBarFix.md) (for lock-screen Now Bar cards)

---

### Step 1: Download and Copy Smart Suggestions

Samsung normally blocks you from updating `Smart Suggestions` manually (it gives a `-3001` error).

First, download **Samsung Smart Suggestions `7.3.30.9`** from APKMirror.
Then send it to your phone using ADB (or just copy it directly in Termux):

```bash
adb push "C:\com.samsung.android.smartsuggestions_7.3.30.9-733009000_minAPI34(arm64-v8a)(nodpi)_apkmirror.com.apk" /data/local/tmp/sss.apk
```

---

### Step 2: Install the APK

We temporarily trick the phone in RAM (`ro.build.official.release false`) so Samsung lets us install the APK without the `-3001` error.

Run this in Termux:

```bash
su -mm -c '
chmod 644 /data/local/tmp/sss.apk
/data/local/tmp/resetprop ro.build.official.release false
SIZE=$(stat -c%s /data/local/tmp/sss.apk)
cat /data/local/tmp/sss.apk | pm install -r -d -i com.sec.android.app.samsungapps -S $SIZE
'
```

---

### Step 3: Unlock Now Brief, Widget, Galaxy AI, and Now Nudge

The app is installed, but the features are still hidden. Samsung checks a system file called `/system/etc/floating_feature.xml` to see if your phone is allowed to use them.

To keep Knox safe, we don't edit the real system file. Instead, we make a copy at `/data/local/tmp/ff_s23.xml`, add the missing flags to it, and mount it over the real file.

Run this in Termux:

```bash
su -mm -c '
umount -l /system/etc/floating_feature.xml 2>/dev/null
cp /system/etc/floating_feature.xml /data/local/tmp/ff_s23.xml

sed -i "s/<SEC_FLOATING_FEATURE_COMMON_CONFIG_AI_VERSION>.*<\/SEC_FLOATING_FEATURE_COMMON_CONFIG_AI_VERSION>/<SEC_FLOATING_FEATURE_COMMON_CONFIG_AI_VERSION>20263<\/SEC_FLOATING_FEATURE_COMMON_CONFIG_AI_VERSION>/g" /data/local/tmp/ff_s23.xml
sed -i "/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_PERSONALIZED_DATA_CORE/d" /data/local/tmp/ff_s23.xml
sed -i "/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_AI_BRIEF_FOR_UT/d" /data/local/tmp/ff_s23.xml
sed -i "/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_NOW_BRIEF/d" /data/local/tmp/ff_s23.xml
sed -i "/SEC_FLOATING_FEATURE_FRAMEWORK_CONFIG_NOW_NUDGE_VERSION/d" /data/local/tmp/ff_s23.xml
sed -i "/SEC_FLOATING_FEATURE_GENAI_CONFIG_LLM_VERSION/d" /data/local/tmp/ff_s23.xml
sed -i "s/<\/SecFloatingFeatureSet>/    <SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_PERSONALIZED_DATA_CORE>TRUE<\/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_PERSONALIZED_DATA_CORE>\n    <SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_AI_BRIEF_FOR_UT>TRUE<\/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_AI_BRIEF_FOR_UT>\n    <SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_NOW_BRIEF>TRUE<\/SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_NOW_BRIEF>\n    <SEC_FLOATING_FEATURE_FRAMEWORK_CONFIG_NOW_NUDGE_VERSION>2<\/SEC_FLOATING_FEATURE_FRAMEWORK_CONFIG_NOW_NUDGE_VERSION>\n    <SEC_FLOATING_FEATURE_GENAI_CONFIG_LLM_VERSION>1.70<\/SEC_FLOATING_FEATURE_GENAI_CONFIG_LLM_VERSION>\n<\/SecFloatingFeatureSet>/" /data/local/tmp/ff_s23.xml

chmod 644 /data/local/tmp/ff_s23.xml
chcon u:object_r:system_file:s0 /data/local/tmp/ff_s23.xml
mount --bind /data/local/tmp/ff_s23.xml /system/etc/floating_feature.xml

/data/local/tmp/resetprop ro.build.official.release false

settings put global sss_enabled 1
settings put global nowbrief_enabled 1
settings put global smart_suggestions_enabled 1
settings put global now_nudge_enabled 1

pm enable com.samsung.android.smartsuggestions/.settings.about.developermode.DeveloperModeActivity
pm enable com.samsung.android.smartsuggestions/.feature.aisuggestion.ui.activity.SuggestionUiContentActivity
pm enable com.samsung.android.smartsuggestions/.feature.aisuggestion.ui.appwidget.AiSuggestionAppWidgetReceiver
pm enable com.samsung.android.smartsuggestions/.feature.aisuggestion.ui.onboarding.OnboardingActivity
pm enable com.samsung.android.smartsuggestions/.brief.BriefSettingsActivity
pm enable com.samsung.android.smartsuggestions/.brief.BriefCustomCardManageActivity
pm enable com.samsung.android.smartsuggestions/.brief.EditableCardActivity
pm enable com.samsung.android.smartsuggestions/.brief.CustomCardCreateActivity
pm enable com.samsung.android.smartsuggestions/.nudge.settings.NudgeMainSettingsActivity

setprop ctl.restart zygote
'
```

**What this command actually does:**
1. Copies `floating_feature.xml` to `/data/local/tmp/ff_s23.xml`.
2. Bumps the AI version from `20253` to `20263` and adds the flags for Personal Data Core, Now Brief, Now Nudge (`2`), and LLM Version (`1.70`).
3. Fixes the file permissions and mounts our edited copy over the original file.
4. Turns on the hidden settings and app screens using `settings put` and `pm enable`.
5. Does a quick soft reboot (`zygote` restart) so the whole phone loads the new features.

---

### What Happens When You Reboot?

Because we didn't touch `/system`, a normal phone reboot will reset the mounted file back to the original one.

After you reboot and get soft root again, you don't need to redo everything. Your `ff_s23.xml` file is already saved, so just run this in Termux:

```bash
su -mm -c '
mount --bind /data/local/tmp/ff_s23.xml /system/etc/floating_feature.xml
setprop ctl.restart zygote
'
```

*(Note: If you also installed **[My FanCam & Audio Eraser](MyFanCam_AudioEraser.md)**, just run `su -mm -c "sh /data/local/tmp/mca/reapply.sh"` instead. It restores everything and restarts `zygote` in one go.)*

---

### Bonus: Notification Highlights (Summarize Notifications)

If you also want **Notification Highlights**, use run these commands and use this repo: [NotiHighlights AI](https://github.com/Furkan-Tsc/Samsung-NotiHighlights-AI).

We already added the required AI flags in Step 3, so you just need to turn the settings on in Termux:

```bash
su -mm -c '
settings put secure noti_intelligence_first_time 1
settings put system ai_info_confirmed 1
settings put secure noti_intelligence_priority_content 1
settings put system noti_intelligence_priority_conversation 1
settings put secure noti_intelligence_summarize_content 1
'
```

| Control Flag | Class / Method | Searching `floating_feature.xml` Tag / Condition | Original Value | Required Value |
| :--- | :--- | :--- | :--- | :--- |
| **General AI Version** | `Rune.COMMON_CONFIG_AI_VERSION` | `SEC_FLOATING_FEATURE_COMMON_CONFIG_AI_VERSION` | `20253` | **`20263`** |
| **Personal Data Engine (PDE)** | `Rune.SUPPORT_PERSONAL_DATA_ENGINE` | `SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_PERSONALIZED_DATA_CORE` | `FALSE` | **`TRUE`** |
| **Now Brief Cards & Settings** | `App.NOW_BRIEF.isPossibleToExpose()` | `SUPPORT_AI_SUGGESTION_UI` and `SUPPORT_PERSONAL_DATA_ENGINE` | `false` | **`true`** |
| **Now Brief UT Support** | `Rune.SUPPORT_AI_BRIEF_FOR_UT` | `SEC_FLOATING_FEATURE_FRAMEWORK_SUPPORT_AI_BRIEF_FOR_UT` | No Value (`FALSE`) | **`TRUE`** |
| **Now Nudge Feature** | `Rune.SUPPORT_NOW_NUDGE` | `SEC_FLOATING_FEATURE_FRAMEWORK_CONFIG_NOW_NUDGE_VERSION` > 0 | No Value (`""`) | **`2`** |
| **Galaxy AI > Now Nudge Menu** | `NowNudgesGalaxyAIController` | `SEC_FLOATING_FEATURE_GENAI_CONFIG_LLM_VERSION` | No Value | **`1.70`** |

#### That's it — enjoy the new features!

<img width="720" height="1482" alt="Screenshot_20261030_231737_Personal data intelligence" src="https://github.com/user-attachments/assets/2cf83b55-a197-4a5e-8ddb-e954822797d9" />
<img width="720" height="1482" alt="Screenshot_20261030_231813_AlwaysOnDisplay" src="https://github.com/user-attachments/assets/b5bf59a0-165b-4064-b5d3-d7616c0e79ba" />
<img width="720" height="1482" alt="Screenshot_20261030_230540_Settings" src="https://github.com/user-attachments/assets/e611e198-5c0f-4a6f-aa76-9ddce998bd6d" />

# TCL 43P7K

TCL's 43" 4K Google TV for Europe (device `G10`, product `G10_4K_GB`, model string "Smart TV Pro",
Realtek `rtd6748`), 2 GB RAM (1.8 GB visible to Android), 16 GB storage (7.9 GB `/data`). Google TV 12,
build `STT2.230203.001`, software `V8-R75PT01-LF1V500`. Remote: Bluetooth TCL RC833 (`BT_RC833A_B5`).

Debloated and running this repo's launcher as Home, set up and checked on a real unit on 2026-10-20.
Nothing is uninstalled; `debloat.txt` lists each disabled package and why. The platform is the one of
[`tcl-g10-google-tv/`](../tcl-g10-google-tv/), one software build newer, and that list applies here too.

All commands run from the repo root with the TV's IP in `TV_HOST`, and need ADB on: Settings > System >
About > press Android TV OS build 7 times, then Developer options > USB debugging, and accept the prompt.
Port 5555 opens straight away; no wireless-debugging pairing was needed. `ro.adb.secure=1`.

## Install

```bash
TV_HOST=<tv-ip> tools/disable.sh examples/tcl-43p7k/debloat.txt
TV_HOST=<tv-ip> HOME_KEY_SERVICE=1 launcher/build-and-install.sh
TV_HOST=<tv-ip> tools/theme.sh <your theme folder>      # the config.json below
```

Set the screensaver before dreamx goes, or it keeps pointing at a disabled package:
`tools/tv.py 'settings put secure screensaver_components com.android.dreams.basic/.Colors'`
(or `com.gathin.tvhome/.ClockDream` for the launcher's clock).

## Undo

```bash
TV_HOST=<tv-ip> tools/restore.sh examples/tcl-43p7k/debloat.txt com.google.android.apps.tv.launcherx
```

## What changed

**Disabled** (see `debloat.txt`): TCL home-screen content and FAST channels, TCL's analytics/updater
package `ttvs`, store demo, e-manual and repair guide, the soundbar updater, Google's preinstall
auto-installer, feedback, TalkBack, Play Movies, Play Games, ambient mode, and the setup wizard
(`setupwraith`), which would otherwise take Home once the stock launcher is off. Then the launcher install
disables `launcherx`.

**Kept on purpose:** Netflix, YouTube, Prime Video and every user-installed app; voice search
(`katniss`) and `tv.remote.service`, which carries the mic button; Chromecast, AirPlay, Miracast;
Game Bar; Live TV/HDMI (`com.tcl.tv`, `com.tcl.tvinput`); the Input button's panel (`com.tcl.suspension`);
the keyboard (`com.tcl.inputmethod.international`, TCL's own IME, so `ttvs`'s T-Cast keyboard is not it).

**Found already disabled** on this unit and left alone: `com.tcl.dashboard` (the stock quick-settings
panel behind the gear key, see below), `com.tcl.factory.view`, `com.tcl.useragreement`, `com.tcl.guard`,
`com.tcl.usercenter`, `com.tcl.messagebox`, `com.tcl.magiconnectfree`, `com.tcl.smartalexa`, `com.tcl.logkit`,
`tv.samba.ssm.ui`, YouTube Music, print spooler, managed provisioning, dynamic system updates. They are not in
the list because they were not checked one by one; `examples/tcl-c755-google-tv` keeps `factory.view` and
`useragreement`.

Verified after each batch and after a reboot: Home, Live TV, voice search, keyboard search, HDMI switching,
Wi-Fi.

## Remote buttons: remapped

With `launcherx` disabled, five buttons did nothing: TCL sends its own keycodes for them, and only the stock
launcher answered. `HomeKeyService` (`HOME_KEY_SERVICE=1`) brings them back from `app_keys` and
`settings_key_keycodes`. Every one was confirmed with the real remote, since injected keys never reach the
service.

| Button | Linux scancode (`getevent`) | Android keycode | Mapped to |
|---|---|---|---|
| Netflix | 242 (`KEY_VIDEO_PREV`, HID `0x0c0051`) | **4062** | `com.netflix.ninja` |
| YouTube | 245 (`KEY_DISPLAY_OFF`, HID `0x0c0057`) | **4065** | `com.google.android.youtube.tv` |
| Prime Video | 752 (`0x2f0`, HID `0x0c005f`) | **4099** | `com.amazon.amazonvideo.livingroom` |
| TV | 365 (`KEY_EPG`) | **4001** | `com.tcl.tv` (Live TV) |
| Settings (gear) | 233 | **83** (`KEYCODE_NOTIFICATION`) | TV settings |

Input keeps working without help: it opens `com.tcl.suspension`'s source panel. The mic works as long as
`tv.remote.service` stays enabled. Do **not** turn on `InputKeyService` as well, because it reads `app_keys`
too and every app button would open its app twice.

The theme used on this unit (`config.json`):

```json
{
  "dock_apps": ["com.netflix.ninja", "com.google.android.youtube.tv", "com.disney.disneyplus", "com.amazon.amazonvideo.livingroom"],
  "hidden_apps": [],
  "live_tv_package": "com.tcl.tv",
  "columns": 5,
  "advanced_effects": true,
  "background_color": "#070A14",
  "wallpaper": true,
  "settings_key_keycodes": [83],
  "app_keys": ["4062:com.netflix.ninja", "4065:com.google.android.youtube.tv", "4099:com.amazon.amazonvideo.livingroom", "4001:com.tcl.tv"]
}
```

with `themes/aurora/wallpaper.jpg` as `wallpaper.jpg`.

## Inputs

The TV input framework publishes only the tuner (`com.tcl.tvinput/.TunerInputService/HW2`). HDMI 1–3 exist as
hardware ports (ids 15–17) and the launcher's Inputs menu opens them through passthrough URIs that `com.tcl.tv`
handles: `content://android.media.tv/passthrough/com.tcl.tvinput/.TvPassThroughService/HW15` (HDMI 1),
`HW16`, `HW17`, and `.../TunerInputService/HW2` for Live TV. No AV row, since the set reports no AV input.

## Resources

Measured with Home on screen after the changes, launcher with wallpaper and advanced effects on:

| | Used | Free |
|---|---|---|
| RAM (1.8 GB) | ~1.13 GB | ~630 MB available |
| Swap (650 MB) | ~172 MB | ~478 MB |
| CPU, 4 cores, Home idle | ~3% | |
| `/data` (7.9 GB) | 2.9 GB | 4.8 GB |
| Launcher (`com.gathin.tvhome`) | 62 MB PSS plain, 65–79 MB with wallpaper and advanced effects; 0% CPU idle | |

What keeps it light: the disabled TCL packages took their boot receivers and background services with them
(`ttvs` alone runs BI, Samba, awareness, Alexa and T-Cast services; `tclhome_passive` starts at boot; ambient
mode keeps fetching pictures). The launcher has no blur, no video and no network polling, and is compiled
ahead of time (`cmd package compile -m speed`). Netflix in the background (154 MB) uses more than anything
the launcher does. No baseline was taken before the changes, so there is no before/after figure.

The load average stays around 25 with the CPU idle. It was the same before any change: on this Realtek
platform it counts driver threads waiting on hardware, not CPU use.

## Firmware quirks

- **Screenshots** come back empty (0 bytes) while Live TV or HDMI is on screen. Check
  `dumpsys activity activities | grep mResumedActivity` instead.
- **Sideloading works.** A first test failed with `INSTALL_FAILED_VERIFICATION_FAILURE`, but only because it
  reinstalled the base APK of a *split* app. Reinstalling a single-APK app (`com.tcl.suspension`) in place
  succeeded, and so did the launcher.
- **Real key presses are not logged.** `WindowManager` logs `handleKeyBeforeQueueing` only for injected keys,
  and `InputReader` logs nothing, so the keycodes above came from a temporary log line in an accessibility
  service. The remote's `.kl` file (`Vendor_0417_Product_0304.kl`) is not readable from the shell, and
  `sendevent` to the remote's node is denied. Run `getevent -lt` through a pty (`adb shell -tt`), or its output
  is lost when it is stopped. After standby, its timestamps run behind `/proc/uptime` by the time the TV
  slept.
- **The TV drops off Wi-Fi** in standby; `adb connect` again after it wakes (no new prompt).
- **`logcat` overruns** a host-side reader within minutes ("significant logging"). Filter on the TV
  (`logcat -s TAG:I`).
- The TV boots into the last source (Live TV) rather than Home; press Home.

# OBS Smooth Zoom
[![Star this Project](https://img.shields.io/badge/Star_this_Repo-f1c40f?style=for-the-badge&logo=github&logoColor=white&labelColor=222)](https://github.com/nextdreambuffer/OBS-Smooth-Zoom-ndb)
![OBS Version](https://img.shields.io/badge/OBS-33.2.2+-blueviolet?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Windows%20|%20macOS%20|%20Linux-lightgrey?style=for-the-badge)

An advanced Lua script for OBS Studio that provides high-quality cursor zoom in real time. This script replicates the polished zoom effects found in paid software like ScreenStudio and Rapidemo.

This script is based on [OBS-Zoom-To-Mouse](https://github.com/BlankSourceCode/obs-zoom-to-mouse) by BlankSourceCode.

> **Community fork by [nextdreambuffer](https://github.com/nextdreambuffer)** — fixes a multi-monitor zoom-follow bug. See [Modifications](#modifications-community-fork) below.

## Demo

https://github.com/user-attachments/assets/dc9c4967-e7f0-4b84-b6c9-e838f11ee4a8

---

## Modifications (Community Fork)

### Why this fork exists

When using this script with a **multi-monitor setup** (primary + secondary display), the zoom-follow feature exhibited two critical bugs that made it unusable on anything other than the primary display:

| Symptom | Root Cause |
|:--------|:-----------|
| **Camera fails to track mouse on the secondary monitor** — zoom center lags or gets stuck at the edge | `get_target_position()` mutated the shared `smoothed_mouse_pos` table in place every frame, corrupting the SmoothDamp starting value |
| **Custom cursor overlay appears in the wrong position on the secondary monitor** — cursor dot drifts from the actual pointer | Cursor overlay was calculated in virtual-screen space while zoom calculations use source-local space |

Both bugs are **invisible on the primary monitor** (offset = 0), which is likely why the original author never encountered them.

### What was fixed

1. **`get_target_position()` no longer mutates its input** (~line 988)
   - Replaced in-place `mouse.x = mouse.x - monitor_info.x` with local copies `mx` / `my`.
   - `smoothed_mouse_pos` is now preserved across frames → SmoothDamp converges correctly.

2. **Cursor overlay now converts to source-local coordinates** (~line 1161)
   - Before the SmoothDamp step, the raw mouse position is offset by the monitor's virtual-screen origin, matching the coordinate space used by `crop_x` / `crop_y`.

3. **`enabled` master toggle** (new)
   - Added a checkbox **☑ Enable Lua** at the top of the Properties panel.
   - When unchecked, all hotkeys and timer callbacks immediately return — the script becomes inert without needing to remove it from OBS.
   - Useful when running **multiple instances** (e.g. testing a fork alongside this one) to avoid conflicts.

### Files changed

- `OBS-Smooth-Zoom-ndb.lua` — two localized coordinate fixes + enable/disable toggle (no breaking API changes)

### Upstream credit

- **Original author**: [JustAdumbPrsn](https://github.com/JustAdumbPrsn) — [OBS-Smooth-Zoom](https://github.com/JustAdumbPrsn/OBS-Smooth-Zoom)
- **Base implementation**: [BlankSourceCode](https://github.com/BlankSourceCode) — [obs-zoom-to-mouse](https://github.com/BlankSourceCode/obs-zoom-to-mouse)

This fork is a minimal patch on top of upstream. If you prefer the unmodified version, use the [original repository](https://github.com/JustAdumbPrsn/OBS-Smooth-Zoom).

---

## Features
* **Smooth Animations:** Smooth transitions for zooming in and out.
* **Dynamic Tracking:** Weighted camera following during active zoom.
* **Custom Cursors:** Support for high-resolution custom cursor and pointer images with Dynamic effects like rotate and tilt
* **Multi-Monitor:** Works with **any number of displays** (2, 3, 4+). The script auto-detects which monitor your Display Capture is pointed at — no config needed. Just point OBS's Display Capture at the target screen.
* **Effect Integration:** Native support for Motion and Zoom blur via external plugins.

---

## Installation

1. **Download the script**
   ```bash
   git clone https://github.com/nextdreambuffer/OBS-Smooth-Zoom-ndb.git
   ```
   or download `OBS-Smooth-Zoom-ndb.lua` directly from this repository.

2. **Load the script into OBS**
   - Navigate to `Tools` → `Scripts`
   - Click the `+` button and select `OBS-Smooth-Zoom-ndb.lua`

3. **Configure a display capture source**
   - Add a `Display Capture` source to your active scene
   - Select the appropriate display from the source properties

4. **Assign hotkeys**
   - Go to `File` → `Settings` → `Hotkeys`
   - Bind keys for:
     - `Toggle zoom to mouse`
     - `Toggle follow mouse during zoom` (optional)

5. **Verify the enable toggle is on** (this fork only)
   - Open the script Properties
   - Ensure **☑ Enable Lua** is checked (it's on by default)
   - If you have multiple instances loaded, only ONE should be enabled

> [!IMPORTANT]
> If you modify Windows display properties (e.g., changing monitor orientation or primary display), you must re-add your `Display Capture` source and reload the script to recalculate coordinates.

> While this script is optimized for Display Capture, you can use Window Capture by enabling `List All Sources (Warning: Performance)` and manually defining source position values.

---

## Optional Configuration

### Custom Cursor Effects

The script supports custom cursor graphics with dynamic animations:

1. **Prepare cursor assets**
   - Download reference assets: `cursor.png` and `pointer.png` from the [original repository](https://github.com/JustAdumbPrsn/OBS-Smooth-Zoom)
   - Or provide your own high-resolution cursor images

2. **Add image sources to OBS**
   - Create two **Image** sources named `cursor` and `pointer`
   - Load your cursor graphics into these sources

3. **Configure in script settings**
   - Navigate to the **Smooth Cursor & Effects** section
   - Assign `cursor` to **Arrow Cursor Source**
   - Assign `pointer` to **Hand Cursor Source (Optional)**
   - Adjust scale and offset parameters to align with your system cursor

4. **Disable native cursor rendering**
   - In your Display Capture source properties, disable `Capture Cursor` to prevent duplicate cursors

### Motion and Zoom Blur Effects

For cinematic blur effects, install the [Composite Blur](https://obsproject.com/forum/resources/composite-blur.1780/) plugin:

1. **Install Composite Blur plugin**
   - Download and install from the OBS plugin repository

2. **Add filters to display capture**
   - Right-click your Display Capture source → `Filters`
   - Add two `Composite Blur` filters
   - Name them exactly: `Zoom Blur` and `Motion Blur`

3. **Configure blur filters**
   - Algorithm: `Gaussian` (quality) or `Box` (performance)
   - Blur Radius: `0.00px` (controlled programmatically by script)

4. **Enable in script settings**
   - Check the corresponding options under **Effects (Blur)**

---

## Compatibility

The following specifications are recommended for optimal performance and stability.

| Component | Specification |
| :--- | :--- |
| **Primary OS** | Windows 11 (Verified) |
| **Secondary OS** | macOS, Linux (Experimental/Community Tested) |
| **OBS Version** | 33.2.2+ |
| **Source Type** | Optimized for `Display Capture` |

---

## Platform Support

While the core interpolation logic is cross-platform, advanced hooks for input detection vary by operating system.

| Feature | Windows | Linux | macOS |
| :--- | :---: | :---: | :---: |
| **Smooth Zooming** | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Partial](https://img.shields.io/badge/-Supported*-brightgreen?style=flat-square) |
| **Mouse Tracking** | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) |
| **Click Animations** | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Not Supported](https://img.shields.io/badge/-Not_Supported-lightgrey?style=flat-square) | ![Not Supported](https://img.shields.io/badge/-Not_Supported-lightgrey?style=flat-square) |
| **Hand/Pointer Detection** | ![Supported](https://img.shields.io/badge/-Supported-brightgreen?style=flat-square) | ![Not Supported](https://img.shields.io/badge/-Not_Supported-lightgrey?style=flat-square) | ![Not Supported](https://img.shields.io/badge/-Not_Supported-lightgrey?style=flat-square) |

### Technical Limitations

* **macOS Architecture:** Smooth zooming functionality on macOS relies on `libobjc`. Users on modern macOS versions may need to verify system permissions for OBS to access the display buffer effectively.
* **Window Capture:** While `Display Capture` is the primary target, other sources can be utilized by enabling **List All Sources (Warning: Performance)** and providing manual coordinate values within the script settings.
* **Linux Environment:** Support is currently focused on X11; Wayland users may experience varying results depending on the compositor’s security policy.
---

### Platform-Specific Notes

* **macOS Support:** Smooth zooming on macOS depends on `libobjc` and specific system permissions. Depending on your version of macOS (notably Sonoma and later), additional security overrides may be required for the script to access the display buffer correctly.
* **Linux Support:** Compatibility is confirmed for X11 environments. Wayland support may vary based on the specific compositor's screen-sharing protocols.
---

## Troubleshooting

### Zoom completely broken / no response (hotkey does nothing)

**Most common cause: two instances of this script loaded simultaneously.**

If you have both `OBS-Smooth-Zoom.lua` (original) and `OBS-Smooth-Zoom-ndb.lua` (this fork) loaded in OBS at the same time, they will conflict:

| Symptom | Why |
|:--------|:----|
| Hotkey does nothing or double-triggers | Both scripts register the same hotkey ID; callbacks overlap |
| Zoom zooms out immediately after zooming in | Two timers fight over the same crop filter |
| Source disappears or shrinks to a tiny dot | Two crop filters stack on the same source |

**Fix — pick one of the following:**

**Option A (recommended): Remove the conflicting script**
1. `Tools` → `Scripts`
2. Select `OBS-Smooth-Zoom.lua` (original) → click **Remove**
3. Keep only `OBS-Smooth-Zoom-ndb.lua`
4. Re-assign your hotkey (`File` → `Settings` → `Hotkeys` → search "Toggle zoom")

**Option B: Use the `enabled` toggle**
1. Keep both scripts loaded (useful when comparing A/B)
2. Open Properties of the script you want to disable
3. **Uncheck** `☑ Enable Lua`
4. The disabled script's callbacks now immediately return — zero side-effects
5. Only the enabled instance responds to hotkeys

> ⚠️ **Warning for forking developers:** If you fork this repo and load both your fork AND `OBS-Smooth-Zoom-ndb.lua` in the same OBS, the same conflict applies. Always disable one via the toggle or remove the other.

### Zoom coordinates are incorrect
- Re-add your Display Capture source after changing monitor configuration
- Reload the script via `Tools` → `Scripts` → Reload

### Custom cursor not appearing
- Verify image source names exactly match: `cursor` and `pointer`
- Check that source visibility is enabled
- Adjust X/Y offsets in script settings

### Blur effects not working
- Confirm Composite Blur plugin is installed
- Verify filter names exactly match: `Zoom Blur` and `Motion Blur`
- Check that filters are applied to the correct source

### Performance issues
- Switch blur algorithm from Gaussian to Box
- Reduce blur radius limits in script settings
- Lower display capture resolution or framerate

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes with descriptive messages
4. Test on your target platform(s)
5. Submit a pull request with a clear description of changes

---

## Acknowledgments

- **Primary author**: [JustAdumbPrsn](https://github.com/JustAdumbPrsn) — [OBS-Smooth-Zoom](https://github.com/JustAdumbPrsn/OBS-Smooth-Zoom)
- **Original concept and base implementation**: [BlankSourceCode](https://github.com/BlankSourceCode) — [obs-zoom-to-mouse](https://github.com/BlankSourceCode/obs-zoom-to-mouse)
- **Multi-monitor fix (this fork)**: [nextdreambuffer](https://github.com/nextdreambuffer)
- Blur integration: [Composite Blur](https://obsproject.com/forum/resources/composite-blur.1780/) by FiniteSingularity

---

## Support

- **Issues (upstream)**: [GitHub Issues](https://github.com/JustAdumbPrsn/OBS-Smooth-Zoom/issues)
- **OBS Forums**: [OBS Project Forums](https://obsproject.com/forum/)

For bug reports, please include:
- OBS version
- Operating system and version
- Monitor layout (number of displays, resolution, relative positions)
- Script settings configuration
- Steps to reproduce the issue

---

## Buy Me a Coffee

If this fork saves you time, consider supporting the development:

<a href='https://ko-fi.com/O0V1270EGQ' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://storage.ko-fi.com/cdn/kofi4.png?v=6' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

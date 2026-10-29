# Focus Visualizer - Master Specification (English)

**Applies to**: v0.9.9.3 Alpha

## 1. Application Overview
A viewer that extracts the metadata recorded by the camera (AF position, subject recognition, shooting info, etc.) from Sony Alpha and Canon EOS RAW images, and overlays the AF frames on the image while staying in sync with Adobe Lightroom Classic.

### Official Repository
- **GitHub**: [https://github.com/muromix/FocusVisualizer](https://github.com/muromix/FocusVisualizer)
- **Release Repository**: [https://github.com/muromix/FocusVisualizer/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest)

### License
- **Personal Use License**: This project is licensed under a proprietary Personal Use Only license. It is free for personal, non-commercial use on your own devices. Redistribution, commercial use, modification, and reverse engineering are prohibited.
- **LICENSE**: Refer to [LICENSE](../LICENSE) in the project root.

### Secure IPC Foundation
Following Electron's recommended settings, the renderer (UI) runs with minimal privileges:
1.  **Context Isolation**: The UI cannot access Node.js / Electron APIs directly.
2.  **Preload Bridge**: Only the channels allowed in `preload.js` are exposed to the UI as `window.api`.
3.  **Delegated File Writes**: The UI never writes files directly; saving is handled by the main process.
4.  **CSP**: Network requests from the UI are restricted to the local server (127.0.0.1).

---

## 2. System Architecture

*   **Lightroom Plug-in (Lua)**: Watches the photo selected in Lightroom and notifies the backend. Writes ratings, labels, and flags set in the viewer back to Lightroom.
*   **Backend (Python / Flask)**: Extracts metadata and preview images with ExifTool and rawpy, and calculates AF frame coordinates.
*   **Viewer (Electron)**: Displays the image and AF frames. See the **[UI Specification (Japanese)](./UI_SPECIFICATION.md)** for details.

**Communication**: HTTP + SSE (127.0.0.1 only)

### 🔐 Security & Privacy
- **Offline by Design**: All photo analysis happens on your own machine. The app does not communicate with external servers.
- **Host-Only Binding**: Communication between Lightroom, the backend, and the viewer is limited to `127.0.0.1` (localhost).
- **No Data Collection**: No image data, browsing history, or personal info is collected or sent.

### Supported Platforms
| OS | Package | How to Launch | Current Release |
| :--- | :--- | :--- | :--- |
| **Windows 10 / 11 (64bit)** | ZIP (runtimes and ExifTool bundled, no installation) | Lightroom: [Library] ＞ [Plug-in Extras] | **v0.9.9.3 Alpha** |
| **macOS (Apple Silicon)** | `Focus Visualizer.app` | Lightroom: [Library] ＞ [Plug-in Extras] | **v0.9.9.3 Alpha** |

---

## 3. Main Features

### 📸 Focus Visualization & Analysis
*   **AF Frames**: Displays the AF position recorded by the camera (toggle with `F`).
*   **Sony Alpha**: In addition to the AF position, displays the type of subject recognition (human, animal, bird, etc.) and the face areas recorded in the maker notes (Tag9401).
*   **Canon EOS**: Displays AF areas from Canon EOS R series metadata.
*   **Focus Peaking**: Detects high-contrast (in-focus) areas in the displayed image and overlays them (`P`). Adjust the threshold with `↑` / `↓`.
*   **Japanese Path Support**: Includes a correction for garbled (mojibake) paths, so photos in folders with Japanese names can be displayed.

### 🔄 Lightroom Sync
*   **Selection Follow**: Selecting a photo in Lightroom switches the viewer. Pressing `←` / `→` in the viewer moves the selection in Lightroom as well.
*   **Write-Back**: Ratings (`1`-`5`), color labels (`6`-`9`, `0`), and flags (Pick / Reject) are applied to Lightroom.
*   **Develop Edits Are Not Reflected**: Crops, rotations, exposure, and other adjustments made in Lightroom are not reflected in the viewer (see Section 4).

### 💎 Quality Control
| Mode | Resolution | Characteristics | Default Status |
| :--- | :--- | :--- | :--- |
| **Speed** | 1600px | Speed first | **Default for both Full / Lite** |
| **Full** | Original | Detailed focus check | Toggle with `Q` |

*   **Browsing Detection**: While `←` / `→` are pressed in quick succession (less than 280ms apart), only the lightweight image is shown regardless of the quality setting, prioritizing navigation speed.
*   **Quality Lock (`L`)**: When enabled in Full quality, moving to the next photo waits until the Full quality image is displayed (released after about 2.5s if there is no response).
*   **Prefetch**: Metadata for neighboring photos is loaded in advance for faster navigation.

### 🪟 Full Mode and Lite Mode
*   **Full Mode**: A window with information panels that opens at the right edge of the screen.
*   **Lite Mode**: A small, always-on-top window. Only `←` / `→`, `Space`, and `Shift + M` are available.
    - **Unlock**: Entering the Konami Code (`↑ ↑ ↓ ↓ ← → ← → B A`) enables rating, label, flag, quality, and other keys in Lite Mode (except Layout Cycling `I`). Enter it again to lock.
*   **Multi-Display Support**: At startup, the window opens on the display where the mouse cursor is. On mode switches and resizing, the window is repositioned within its current display.

---

## 4. Core Philosophy: Selection Specialized
This project does not aim to be a generic image editor. It is a specialized tool for **culling** — selecting the best shots quickly from a large number of RAW files.

1.  **Check the Image as the Camera Recorded It**:
    Crops, rotations, exposure, and other adjustments made in Lightroom are not reflected. AF frames are overlaid on the image as recorded by the camera.
2.  **Decision Support**:
    Beyond displaying the image, the viewer shows the AF position and subject recognition state (face, animal, bird, etc.) so you can judge focus without hesitation.
3.  **Independence & Agility**:
    Instead of waiting for Lightroom's preview generation, images and metadata are extracted directly from the RAW file (ExifTool / rawpy).

---

## 5. Keyboard Shortcuts
| Key | Action |
| :--- | :--- |
| `Space` | **Target Zoom** (Fit ↔ 1:1; at 1:1 the focus point is centered) |
| Mouse wheel / Drag / Double-click | Zoom / Pan / Back to Fit |
| `←` / `→` | Image Navigation (Lightroom linked) |
| `1` - `5` | Rating (press the same number again to clear) |
| `6` / `7` / `8` / `9` / `0` | Color Label (Red / Yellow / Green / Blue / Purple) |
| `Backspace` | Clear Color Label |
| `A` / `X` / `U` | Flag (Pick / Reject / None) |
| `Q` | Quality Mode Toggle (Speed ↔ Full) |
| `L` | Quality Lock |
| `F` | AF Frames ON/OFF |
| `P` | **Peaking** ON/OFF |
| `↑` / `↓` | Peaking Threshold |
| `I` | Layout Cycling (Full Mode only) |
| `M` / `Shift + M` | **Mode Toggle** (Full ↔ Lite) |

*Note: In Lite Mode, only `←` / `→`, `Space`, and `Shift + M` are available until unlocked.*

---

## 6. Changelog
See [Releases](https://github.com/muromix/FocusVisualizer/releases) for the changes in each version.

---

**Created by muromix with Angie**
"Code is the ultimate love letter to our future selves."

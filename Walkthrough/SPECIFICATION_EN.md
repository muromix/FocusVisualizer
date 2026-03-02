# Focus Visualizer - Master Specification (English)

## 1. Application Overview
A full-stack viewer that extracts metadata (focus position, model info, etc.) from RAW images (ARW) captured with Sony Alpha series (like a7R V), providing high-precision visualization while maintaining real-time synchronization with Lightroom.

### Official Repository
- **GitHub**: [https://github.com/muromix/FocusVisualizer](https://github.com/muromix/FocusVisualizer)
- **Release Repository**: [https://github.com/muromix/FocusVisualizer/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest)

### License
- **MIT License**: This project is released under the MIT License. Free use, modification, and redistribution are allowed for both commercial and non-commercial purposes, provided the copyright notice and disclaimer are retained.
- **LICENSE**: Refer to `LICENSE.txt` in the project root for details.

### 1.0 Session Initialization Protocol - *CRITICAL*
At the start of a session, the AI must prioritize reading the following files as the "Source of Truth":
1.  **`DevStory/index.md`**: Overall progress history.
2.  **`DevStory/README.md`**: Philosophy and structure of the dev story.
3.  **`./SPECIFICATION.md`**: This document (Master Specification).
4.  **Latest report in `DevStory/` folder**: Most recent tasks and issues.
5.  **`./METADATA_SCHEMA.md`**: Definition of **UIMS v1.0 (Common Language)**.
6.  **`./BACKEND_MODULARIZATION_PROMPT.md`**: Blueprint for backend refactoring.
7.  **`Dev/_repomix_all_packs/`**: Sync source code of the entire project.

### 1.1 Secure IPC Foundation - *Introduced 2026/02/23*
Minimizing renderer process privileges based on Electron's latest best practices.
1.  **Context Isolation**: Blocks renderer (`renderer.js`) from direct access to Node.js/Electron APIs.
2.  **Preload Bridge**: Exposes only trusted APIs via `window.api` in `preload.js`.
    - `window.api.send / on / invoke`: Intermediates only allowed IPC channels.
    - `window.api.path`: Executes system path operations safely on the main process side.
3.  **File System Control**: Abolishes direct file writing (`fs.writeFileSync`) from the renderer.
4.  **CSP (Content Security Policy)**: Restricts requests to the internal server (127.0.0.1) only.

## 2. System Architecture
Refer to individual documents for detailed technical specifications.

*   **[Lightroom Plugin (Lua)](../1.Plugin_for_Lightroom)**: Monitors photo selection events.
*   **[Python Backend](../3.Python_Backend)**: **[Technical Detail](./BACKEND_PYTHON_SPECIFICATION.md)** - Type-safe analysis based on Pydantic v2 & UIMS v1.0.
*   **[Focus Point Viewer (UI)](../2.Viewer_App)**: **[UI Detail](./UI_SPECIFICATION.md)** - Preview display and AF frame rendering.

**Platform**: Windows / macOS
- **Backend**: Python (Flask) - Modular Service Architecture (DI)
- **Frontend**: Electron (High-speed Canvas rendering)
- **Communication**: HTTP + SSE (Strictly bound to 127.0.0.1)

### 🔐 Security & Privacy
- **Offline by Design**: Strictly for local use. No communication with external servers.
- **Host-Only Binding**: All internal communication restricted to `127.0.0.1` (localhost).
- **No Data Collection**: No image data or personal information is collected.
- **License**: MIT License.

### 2.1 Supported Platforms - *Updated 2026/03/01*
| OS | Backend | ExifTool | Launch Method | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Windows 10/11** | **Portable Python (Bundled)** | Vendor Bundled | **`Launcher.vbs` (Stealth)** | ✅ **Industrial Grade (v0.9.5_Alpha)** |
| **macOS** | **focus_backend (onedir)** | **System Priority** | **Focus Visualizer.app** | ✅ **Full Support (v0.9.5_Alpha)** |

---

## 3. Main Features

### 📸 Focus Visualization & Analysis
*   **High-Speed Display**: Under 0.3s display speed for RAW images via embedded preview extraction.
*   **Focus Visualization**: Green AF frames, tracking status (a7R V), and subject recognition (Eye/Face).
*   **Focus Peaking**: Visualizes focus areas with dots. Automatic sensitivity adjustment based on Quality Mode.
*   **Shutter Speed Precision**: Prioritizes camera setting values (e.g., 1/40) over effective exposure time.
*   **File Quality**: Analyzes and displays RAW compression formats and sizes from Exif.
*   **Japanese Path Support (Mojibake Rescue)**: Automatically repairs charset issues on Windows.

### 🔄 Lightroom Sync
*   **Bi-directional Navigation**: Instant sync of photo navigation between the viewer and Lightroom.
*   **Rating & Label Sync**: Immediate application of ratings, color labels, and flags to Lightroom.

### 💎 Quality Mode - *Updated 2026/02/23*
| Mode | Resolution | Characteristics | Behavior |
| :--- | :--- | :--- | :--- |
| **Speed** | 1600px | **Ultra-Fast & Simple** | **Default**. 40ms throttle for snappy operation. |
| **Full** | Original | Ultimate Focus Check | **Auto-zoom to focus point** at 100% scale upon switching. |

---

## 4. Keyboard & Mouse Interaction

### 4.1 Keyboard Shortcuts
| Key | Category | Action |
| :--- | :--- | :--- |
| `Space` / `Shift` | Zoom | **Precision Target Zoom** (100% ↔ Fit) |
| `←` / `→` | Nav | Photo Navigation (Previous / Next) |
| `Q` | Quality | Quality Mode Toggle (Speed ↔ Full) |
| `L` | Quality | Quality Lock (Manual override) |
| `P` | Tool | **Peaking** ON/OFF |
| `+` / `-` / `↑` / `↓` | Tool | Peaking Sensitivity Adjustment |
| `F` | Display | Focus Frame Toggle |
| `I` | Display | Layout Cycle (Wide ➔ Mid ➔ Compact) |
| `A` / `X` / `U` | Sync | Flagging (Pick / Reject / Unflag) |
| `1` - `5` | Sync | Rating Setting (1-5 Stars) |
| `6` - `9` / `0` | Sync | Color Label Setting |

---

## 5. Technical Details & Communication
*   **HTTP (Port 8765〜)**: Command and data communication.
*   **SSE `/stream` (HTTP)**: Real-time push notifications from backend to frontend.

---

## 6. Coordinate Transformation
1.  **Raw Coordinates (1000-based)**: Normalized coordinates from 0 to 1000.
2.  **Preview Coordinates**: Mapping to the image's physical size (with Orientation).
3.  **Canvas Coordinates**: Final conversion to the display size (Fit / Zoom).

---

## 7. UI/UX Design Specification
"Side View" optimized for mobile environments (e.g., 13-inch MacBook Pro).
- **Intelligent Auto-Resize**: 3-step layout [Wide / Mid / Compact].
- **Placement**: Snaps to the right edge of the primary screen.

---

## 8. Deployment & Sync Protocol
Strict management of assets when syncing from the development environment (`Dev/`) to the distribution environment (`dist/`).

---

## 9. Changelog
- **2026/03/02**: **"The Open Wings"** (Migration to MIT License, extreme build optimization)
- **2026/03/01**: **"v0.9.5 Alpha: Zero-Friction Milestone"** (Portable Python, VBS Stealth Launcher)

---

**Created by muromix with Lead Engineer & Angie**
"Code is the ultimate love letter to our future selves."

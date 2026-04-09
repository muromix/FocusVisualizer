# Focus Visualizer - Master Specification (English)

## 1. Application Overview
A high-performance full-stack viewer that extracts metadata (focus position, model info, etc.) from Sony Alpha (e.g., A7R V) and Canon EOS (e.g., R5, R6 III) RAW images, visualizing it with high precision while maintaining real-time synchronization with Adobe Lightroom Classic.

### Official Repository
- **GitHub**: [https://github.com/muromix/FocusVisualizer](https://github.com/muromix/FocusVisualizer)
- **Release Repository**: [https://github.com/muromix/FocusVisualizer/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest)

### License
- **Personal Use License**: This project is licensed under a proprietary Personal Use Only license. It is free for personal, non-commercial use on your own devices. Redistribution, commercial use, modification, and reverse engineering are prohibited.
- **LICENSE**: Refer to [../LICENSE.md](../LICENSE.md) in the project root.

### Secure IPC Foundation
Based on modern Electron best practices:
1.  **Context Isolation**: Secure bridge between `renderer.js` and Node.js API to prevent global namespace pollution.
2.  **Preload Bridge**: Only trusted APIs exposed via `window.api`.
3.  **System Lock**: Direct file writes are prohibited from the renderer; handled proxy-style by the main process via IPC.
4.  **CSP**: Strictly limited to local server (127.0.0.1) requests to prevent unexpected external communication.

---

## 2. System Architecture
Please refer to individual documents for detailed technical specs.

*   **[Lightroom Plugin (Lua)](../1.Plugin)**: Event monitoring and sync.
*   **[Python Backend](../3.Python_Backend)**: **[Technical Docs](./BACKEND_PYTHON_SPECIFICATION.md)** - Pydantic v2 & UIMS v1.0.
*   **[Focus Point Viewer (UI)](../2.Viewer)**: **[Technical Docs](./UI_SPECIFICATION.md)** - Canvas rendering.
*   **[4-Layer Metadata Fortress](./METADATA_FORTRESS_ARCHITECTURE.md)**: Architectural design for governing asynchronous consistency.

**Platform**: Windows / macOS
- **Backend**: Python (Flask) - Modular Service Architecture (DI)
- **Frontend**: Electron (High-speed Canvas rendering)
- **Communication**: HTTP + SSE (127.0.0.1 bound)

### 🔐 Security & Privacy
- **Offline by Design**: This app is strictly for local use. No cloud processing.
- **Host-Only Binding**: Internal sync via `127.0.0.1` (localhost) only.
- **No Data Collection**: No image data or personal info is collected.

---

## 3. Main Features

### 📸 Focus Visualization & Analysis
*   **High Speed**: RAW image preview in under 0.3s.
*   **AF Coordinates**: Renders green AF boxes and tracking states.
    - *(Sony AI Recognition / Tag9401 parsing is currently paused for further refinement)*
*   **Focus Peaking**: Contrast-detection based real-time peaking. 50ms latency.
*   **Mojibake Rescue**: Native handling for non-ASCII (Japanese) file paths on Windows.

### 🔄 Lightroom Sync
*   **Two-Way Sync**: Instant reflection of ratings (0-5), color labels (6-9), and flags (Pick/Reject).

### 💎 Quality Control
| Mode | Resolution | Characteristics | Default Status |
| :--- | :--- | :--- | :--- |
| **Speed** | 1600px | **Fast & Smooth** | **Default for both Full/Lite** |
| **Full** | Original | Precision culling | Manual / Q-key only |

*   **Lite Mode Evolution (Power-Up)**: 
    - **Full Shortcut Unlock**: Power users (Unlocked via Konami Code) can now use all shortcuts (Rating, Label, Flag, Quality) in Lite mode (except for Layout Cycling `I`).
    - **GUI Minimalism**: Mode switching is now handled strictly via shortcuts (`M`). The transition button is hidden even when unlocked to maintain a distraction-free environment.
    - **Performance**: Defaults to **Speed Mode** (1600px) to prioritize record-speed navigation.

*   **Multi-Display Adaptation (Display Sticky)**: 
    - **LRC-Sticky Placement**: At startup, detects the display containing the cursor (where the user is active in LRC) and places the window there.
    - **Monitor Locking**: Mode switching and resizing are anchored to the current display (detected via `screen.getDisplayMatching()`), preventing accidental window jumps between monitors.

*   **Quality Lock & Blocking**: 

## 4. Core Philosophy: Selection Specialized - *Defined 2026/03/05*
This project does not aim to be a generic image editor. It is a specialized professional tool for **"Culling" (selecting the best shots at record speed).**

1.  **"True Development" Path**: 
    We intentionally do not replicate secondary edits like Lightroom's manual crops or exposure compensation. Instead, we perform "Pure Development" (rawpy) to ensure 100% coordinate precision between maker metadata and pixels.
2.  **Cognitive Decision Support**: 
    Beyond just displaying an image, we visualize AF states (eyes, face, tracking) to help users make "instant, confident" decisions.
3.  **Independence & Agility**: 
    We bypass Lightroom's heavy preview generation, using our own high-speed extraction engine to accelerate the culling workflow.

---

## 5. Keyboard Shortcuts
| Key | Action |
| :--- | :--- |
| `Space` | **Precision Target Zoom** (1:1 ↔ Fit) |
| `←` / `→` | Image Navigation (LRC Linked) |
| `Q` | Quality Mode Toggle |
| `L` | Quality Lock |
| `P` | **Peaking** ON/OFF |
| `A` / `X` / `U` | Flagging (Pick / Reject / Unflag) |
| `1` - `5` | Set Ratings |
| `M` | **Mode Toggle** (Full ↔ Lite) |

*Note: Unlocked power users can use all of the above shortcuts (except 'I') in Lite Mode.*

---

## 9. Changelog Highlights
- **2026/03/31**: **"Minimalism & Multi-Display Sticky Behavior"**
  - **Lite Mode Power-Up**: Unlocked Lite Mode via Konami code now allows all selection shortcuts (1-9, Q, A, X, U) to be processed.
  - **GUI Minimalism**: Removed the mode promotion button in favor of a strictly shortcut-driven (M key) workflow.
  - **Smart Monitor Mapping**: Introduced display-aware positioning (`screen.getDisplayMatching`). The viewer now opens on the LRC monitor and follows the current screen during mode swaps.
  - **Zen Design Evolution**: Instead of a complex "Auto-Zen" mode, we've simplified the existing Lite Mode to provide a pure, immersive culling experience.
- **2026/03/13**: **"4-Layer Metadata Fortress (Metadata Persistence Mastery)"**
  - **Metadata Immutability**: Solved the "vanishing ratings" bug during quality switches or browser reloads.
  - **4-Tier Defense**: Implemented a coordinated architecture (Heartbeat Guard, Action Guard, Injection Guard, and Authority Guard) to govern asynchronous race conditions.
  - **Metadata Sync Back-Channel**: Developed `request_metadata` logic via SSE for automatic recovery when backend state is stale.
  - **AI Strategy Orchestration**: Established a zero-cost "Dual-Brain" workflow using Web-based Gemini and latest Gemini 2.5/3.1 SDKs.
- **2026/03/11**: **"Solid Frame Navigation & Portrait Orientation Unification"**
  - **One-at-a-Time Navigation**: Implemented render-gated navigation. Arrow keys are now blocked until the current image is fully rendered, eliminating race conditions and wasted resources during rapid switching.
  - **EXIF Stripping (All Paths)**: Unified `_strip_exif_marker()` across ALL preview extraction paths (ExifTool embedded, rawpy FULL, DNG balanced). Portrait images now display correctly at every quality level.
  - **Cache Path Bug Fix**: Fixed SSE push missing `data.path` on cache-hit responses, which caused the frontend to silently skip valid image update events.
  - **Planned: Browsing Mode (Phase 1-3)**: A new `isBrowsingMode` detection system (280ms threshold) is planned. During rapid key-hold, only SPEED preview renders. On pause, FULL quality upgrades silently.

- **2026/03/06**: **"Flipbook Paging & Turbo Shift (Extreme Speed Optimization)"**
  - **Flipbook Navigation**: Implemented "Smart Catch-up" logic. If the processing queue builds up (> 2), it automatically skips to the latest request for zero-latency response.
  - **Metadata Debouncing**: Introduced a 100ms settle delay for ExifTool analysis to prevent CPU spikes during rapid navigation.
  - **Launcher Reliability**: Batch launchers are now non-blocking (`start` command) to prevent Lightroom freezes. Added UTF-8 logging to `Logs/launcher_debug.txt`.

- **2026/03/05**: **"The AI Captured & Snappy Update (AI Research Phase)"**
  - **Sony AI Recognition Research**: Experimental binary parsing for `Sony:Tag9401` and SubjectRecognition logic (currently under further research/refinement).
  - **AI Visual Overlay**: Prototyped white frames for AI-tracked subjects.
  - **Zero-Flicker Architecture**: Implemented a 300ms metadata wait-loop to eliminate vertical rotation flickering.
  - **Lite Mode Speed Boost**: Defaulted initial quality to "Speed" (1600px) for rapid navigation.

- **2026/04/09**: **"License Lockdown (Personal Use Only)"**
  - **License Enforcement**: Transitioned from open-source (MIT) to a proprietary Personal Use Only license to protect muromix's intellectual property. Redistribution and commercial use are now prohibited.

- **2026/03/02**: **"Open Source Pilot"**
  - Briefly piloted the project under the MIT License.
  - **Payload Reduction (158MB)**: Fixed recursive build bug (mirror-in-mirror).
  - **Mac Launch Boost**: `--onedir` migration for faster backend startup.

---

**Created by muromix with Angie**
"Code is the ultimate love letter to our future selves."

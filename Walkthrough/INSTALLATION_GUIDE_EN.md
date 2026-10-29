# Focus Visualizer - Installation & Setup Guide (v0.9.9.3 Alpha)

**Created by muromix & Angie**

This document provides instructions for installing and setting up Focus Visualizer for Adobe Lightroom Classic (LRC).
Updated for the **v0.9.9.3 Alpha** release.

---

## 🚀 1. System Requirements

The Windows version is a **portable product**—no separate installation of Python or Node.js is required.

### 🪟 For Windows Users
- **OS**: Windows 10 / 11 (64bit)
- **Dependencies**: None (All runtimes are bundled in the package)
- **Lightroom Classic**: v12.0 or higher recommended

### 🍎 For Mac Users
- **Mac**: Apple Silicon Mac
- **OS**: macOS 13 (Ventura) or higher
- **Package**: `v0.9.9.3_Alpha_Mac.zip` (not notarized by Apple; see "Setup Instructions (Mac)" below for the first launch)

---

## 📦 2. Setup Instructions (Windows)

The Windows version is designed to be "unzip and run".

1. Extract the downloaded ZIP archive and place the folder anywhere you like (for example, a `plugin` folder you create in your user folder).
   - Avoid non-ASCII characters (such as Japanese) and special symbols in the folder path; they may prevent the tool from working. If your Windows user name contains such characters, place the folder outside your user folder.
2. Open **Adobe Lightroom Classic**.
3. Go to **[File] ＞ [Plug-in Manager]**.
4. Click the **[Add]** button and select the **`1.Plugin_for_Lightroom/Focus_Visualizer.lrplugin`** folder inside the extracted directory.
5. The status should show that the plug-in is enabled.

### Setup Instructions (Mac)

1. Extract the downloaded ZIP and move the whole `FocusVisualizer_v0.9.9.3_Alpha_Mac` folder to your Applications folder (or anywhere you like). The app, the plug-in, and the documents are all in this folder.
2. **First launch only**: Double-click `Focus Visualizer.app` in the folder. When the warning appears, close it, open **[System Settings] ＞ [Privacy & Security]**, and click **[Open Anyway]** next to the message that the app was blocked. If you are asked for access to folders such as Desktop, allow it.
3. In Lightroom Classic, go to **[File] ＞ [Plug-in Manager]**, click **[Add]**, and select **`Plugins/Focus_Visualizer.lrplugin`** inside the folder.
4. If you move the folder after registering, register the plug-in again.

More details are in the included `README (EN).txt`.

---

## 🎯 3. How to Launch (via LRC)

Focus Visualizer is designed to be launched directly from within Lightroom Classic.

1. Select a photo in the Library module.
2. Go to **[Library] ＞ [Plug-in Extras] ＞ [Focus Visualizer]**.
3. The viewer application will launch automatically and visualize the focus points of the selected photo.
   - On Windows, a command prompt window may appear briefly at startup.

---

## 🛠️ 4. Maintenance & Troubleshooting

- **Unified Logging (`Logs/`)**:
  Logs from the backend, Electron, and the Lightroom plug-in are written to the **`Logs/`** directory directly under the extracted folder.
  - On Mac, the backend and viewer logs are written to `~/Library/Application Support/Focus Visualizer/Logs` (the Lightroom plug-in log is in the `Logs/` folder inside the package folder).
- **Uninstall**:
  The application does not modify your system registry. Remove it from the Lightroom Plug-in Manager and delete the application folder.
  - On Windows, a small record (such as the last displayed photo) remains in `%APPDATA%\Focus Visualizer`. Delete this folder as well if you no longer need it.
  - On Mac, a similar record and the logs remain in `~/Library/Application Support/Focus Visualizer`.

---

## ☕ Support the Project

Focus Visualizer is a personal project built to accelerate the workflow of photographers (free for personal use).
If this tool saves you time, please consider supporting our continued development:

- **[Support via BOOTH (Angie's Lab)](https://angieslab.booth.pm/)**
- **[Buy Me a Coffee](https://buymeacoffee.com/muromix)**

---
*Focus Visualizer - Developed by muromix & Angie.*

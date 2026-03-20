# Focus Visualizer - Installation & Setup Guide (v0.9.8 Alpha)

**Created by muromix & Angie**

This document provides instructions for installing and setting up Focus Visualizer for Adobe Lightroom Classic (LRC).
Updated for the **v0.9.8 "SUPER SLIM"** release.

---

## 🚀 1. System Requirements

The Windows version is now a **strictly portable product**—no separate installation of Python or Node.js is required.

### 🪟 For Windows Users
- **OS**: Windows 10 / 11 (64bit)
- **Dependencies**: None (All runtimes are bundled in the package)
- **Lightroom Classic**: v12.0 or higher recommended

### 🍎 For Mac Users
- **OS**: macOS 12.0 (Monterey) or higher recommended
- **Current Version**: Mac version is currently at **v0.9.5 Alpha** (v0.9.8 parity update coming soon).

---

## 📦 2. Setup Instructions (Windows)

The Windows version is designed to be "unzip and run".

1. Extract the downloaded ZIP archive to a location of your choice.
2. Open **Adobe Lightroom Classic**.
3. Go to **[File] ＞ [Plug-in Manager]**.
4. Click the **[Add]** button and select the **`1.Plugin_for_Lightroom`** folder from the extracted directory.
5. The status should show a green circle indicating the plug-in is active.

---

## 🎯 3. How to Launch (via LRC)

Focus Visualizer is designed to be launched directly from within Lightroom Classic.

1. Select one or more photos in the Library module.
2. Go to **[File] ＞ [Plug-in Extras] ＞ [Focus Visualizer for Sony Alpha]**.
3. The viewer application will launch automatically and visualize the focus points for the selected images.

---

## 🛠️ 4. Maintenance & Troubleshooting

- **Unified Logging (`Logs/`)**: 
  All system logs (Backend, Electron, and Lightroom Plugin) are consolidated into the **`Logs/`** directory at the project root for easier troubleshooting.
- **Portability**: 
  The application does not modify your system registry. To uninstall, simply remove it from the Lightroom Plug-in Manager and delete the application folder.

---

## ☕ Support the Project

Focus Visualizer is a passion project built to accelerate the workflow of photographers.
If this tool saves you time, please consider supporting our continued development:

- **[Support via BOOTH (Angie's Lab)](https://angieslab.booth.pm/)**
- **[Buy Me a Coffee](https://buymeacoffee.com/muromix)**

---
*Focus Visualizer - Developed by muromix & Angie.*

# Focus Visualizer - Installation & Setup Guide

This document provides instructions for installing and setting up Focus Visualizer for Adobe Lightroom Classic.

## 1. System Requirements

### For Windows Users
- **OS**: Windows 10 / 11 (64bit)
- **Python**: Version 3.10 or higher (Must be installed on your system)
- **Lightroom Classic**: v12.0 or higher recommended
- **Memory**: 8GB+ (16GB+ recommended)

### For Mac Users
- **OS**: macOS 12.0 (Monterey) or higher recommended
- **Node.js**: v18 LTS or higher recommended ([Download](https://nodejs.org/))
- **Python**: Version 3.10 or higher recommended (`setup_mac.command` will check this automatically)
- **ExifTool / Homebrew**: Will be installed automatically by `setup_mac.command` if not present.

---

## 2. Installation & Setup

### 🪟 Windows Setup

1. Extract the downloaded `Focus_Visualizer` folder to a location of your choice.
2. Double-click **`setup_win.bat`** located in the root folder.
   *(This will automatically create a Python virtual environment and install required libraries).*

### 🍎 Mac Setup

1. Double-click **`setup_mac.command`** located in the root folder.
   *(If you see a security warning: Right-click -> [Open] -> [Open])*
2. The script will automatically perform the following:
   - Check and install Homebrew
   - Install ExifTool via brew
   - Install UI dependencies (`npm install`)
   - Create Python venv & install backend libraries
   - Grant execution permissions
3. Setup is complete when you see the message "🎉 Setup Complete!".

---

## 3. Lightroom Classic Integration

This step is common for both Windows and Mac.

1. Open Adobe Lightroom Classic, go to **[File] > [Plug-in Manager]**.
2. Click the **[Add]** button and select the `1.Plugin/Focus_Visualizer.lrplugin` folder from the extracted directory.
3. Once registered, the setup is complete! You can now launch it anytime by selecting a photo and going to **[File] > [Plug-in Extras] > [Focus Visualizer for Sony Alpha]**.

---

## 4. Uninstallation

This application does not modify your system registry or core settings. To uninstall safely:

1. Open Lightroom Classic, go to **[Plug-in Manager]**, select "Focus Visualizer for Sony Alpha", and click **[Remove]**.
2. Delete the downloaded `Focus_Visualizer` folder (Move to Trash/Recycle Bin).
3. If you no longer need Python or Node.js, you can manually uninstall them from your OS settings.

---
*Focus Visualizer - Developed by muromix & Angie.*

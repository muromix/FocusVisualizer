# 🎯 Focus Visualizer v0.9.5 Alpha: Technical Operations Guide

This document provides a detailed technical overview of **Focus Visualizer**. It is intended for users who want to understand the features, requirements, and installation process of the alpha release.

---

## 🎯 Tool Overview
Focus Visualizer is a high-speed workflow utility for photographers using Adobe Lightroom Classic. It extracts and displays hidden metadata that Lightroom natively ignores:
- **AF Point Visualization**: Displays the exact green AF coordinates and tracking target (Eye/Face) used at the moment of capture. **Supports Sony Alpha and Canon EOS series.**
- **Deep Metadata**: Camera data, focus distance, and RAW quality details.
- **LRC Two-Way Sync**: Synchronizes Ratings (1-5), Color Labels, and **Flags (Pick/Reject)** with Lightroom in real-time.

---

## ✨ Updates & Key Features (v0.9.5 Alpha)
- **Expanded Camera Support**: Initial support for **Canon EOS series** (R6 Mark III, 7D Mark II, etc.) has been added alongside Sony Alpha. *(Note: Canon support for Mac is being finalized!)*
- **Portable Runtime (Windows)**: For Windows users, the application is now fully portable. **No separate installation of Python or Node.js is required.**
- **Precision Target Zoom**: Pressing [Space] creates a 100% crop centered precisely on the extracted focus coordinates.
- **Focus Peaking**: Pressing [P] visualizes in-focus areas using contrast-detection dots (computed in under 50ms).
- **Session Persistence**: The app automatically remembers your last viewed image and quality settings.
- **Seamless Navigation**: Use the arrow keys in the Viewer to switch photos, and your Lightroom selection will instantly follow along.
- **Stealth Background (Win)**: ExifTool now runs silently without popping up command prompt windows.
- **100% Offline / Local Execution**: The tool operates strictly on `localhost`. No external requests or data collection are performed.

---

## 💻 System Requirements
- **OS**: macOS (Apple Silicon) OR Windows 10 / 11 (64bit)
- **Adobe Lightroom Classic** (v12.0+)
- **System**: 8GB RAM minimum, SSD required for optimal preview extraction speed.
*Note: The Windows version includes its own internal runtime. No separate installation of Python or ExifTool is required.*

---

## 📥 Installation
1. Download the archive for your OS (**ZIP** for Windows, **DMG** for Mac).
2. Follow the instructions in **`INSTALL_EN.txt`** (located inside the folder).
3. If you encounter bugs, please report them via the GitHub **Issues** tab.

---

## ⚖️ License & Disclaimer
**This software is released under the [MIT License](LICENSE).**

Copyright (c) 2026 muromix.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---

## ☕ Support the Project
Focus Visualizer is a labor of love by individual developers and AI partners. If you find this tool useful, consider supporting the development!

[![Buy Me a Coffee](https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20coffee&emoji=☕&slug=muromix&button_colour=FF5F5F&font_colour=ffffff&font_family=Cookie&outline_colour=000000&coffee_colour=FFDD00)](https://buymeacoffee.com/muromix)

*(c) 2026 muromix | Synergy of Human and Machine Intelligence.*

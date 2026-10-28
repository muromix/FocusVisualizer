# Focus Visualizer - Knowledge Portal
# Focus Visualizer - ナレッジ・ポータル

This portal connects the concept, specifications, and setup guides of Focus Visualizer.
Focus Visualizer の理念・仕様・導入手順をまとめた入口です。

---

## 🗺️ System Overview / システム俯瞰図

```mermaid
graph TD
    subgraph "Lightroom Classic (LRC)"
        LUA["Plugin (Lua)"]
    end

    subgraph "Focus Engine (Backend)"
        B_CORE["Python Service (Flask)"]
        B_IP["Image Processor (rawpy)"]
        B_MP["Meta Parser (ExifTool)"]
    end

    subgraph "Visualizer (Frontend)"
        F_EL["Electron App"]
        F_CV["Canvas Renderer"]
    end

    LUA -- "Photo selection / polling (HTTP POST, 127.0.0.1)" --> B_CORE
    B_CORE -- "Rating / Label / Flag actions (poll response)" --> LUA
    B_CORE -- "Metadata & AF coordinates (SSE)" --> F_EL
    B_CORE -- "Preview image (HTTP)" --> F_EL
    F_EL -- "Actions (Rating / Label / Flag / Navigation)" --> B_CORE
    B_MP -- "Metadata" --> B_CORE
    B_IP -- "Preview" --> B_CORE
    F_EL --> F_CV
```

---

## 🌐 Language Selection / 言語選択
The Master Specification is maintained in both English and Japanese.
全体仕様書は、日英両言語で用意しています。

- 🇺🇸 **[English Master Spec (SPECIFICATION_EN.md)](./SPECIFICATION_EN.md)**
- 🇯🇵 **[日本語 全体仕様書 (SPECIFICATION_JP.md)](./SPECIFICATION_JP.md)**

---

## 🗂️ Asset Directory / 資産ディレクトリ

### 1. Vision & Concept / 理念とコンセプト
- [Concept & Philosophy](./FOCUS_VISUALIZER_CONCEPT.md) - Why we build this.

### 2. Viewer / ビューアー
- [UI Specification (Japanese)](./UI_SPECIFICATION.md) - Layout, display logic, and window behavior.

### 3. EXIF Reference / EXIF 資料
- [EXIF Manual (Japanese)](./EXIF_TAGS_GUIDE_JP.md) - User-friendly tag list (Sony α7R V sample).

### 4. Setup & Releases / 導入とリリース
- [Installation Guide (EN)](./INSTALLATION_GUIDE_EN.md) - Setup for English users.
- [インストールガイド (JP)](./INSTALLATION_GUIDE_JP.md) - 日本語環境向けの手順。
- [Releases](https://github.com/muromix/FocusVisualizer/releases/latest) - Downloads and release notes.

---

**Created and Maintained by muromix & Angie**
"Code is the ultimate love letter to our future selves."
「コードは、未来の自分たちへの究極のラブレターよ。」

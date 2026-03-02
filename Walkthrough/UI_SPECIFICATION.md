# Sniper Sync - Viewer UI Specification (v3.0)
# Sniper Sync - ビューアーUI仕様書 (v3.0)

**Date**: 2026-02-22 (Refactored to match v0.9.0 Alpha structure)
**作成日**: 2026-02-22 (v0.9.0 Alphaの実際の構造に合わせて再定義)

This document defines the UI component hierarchy, visual layout, and interaction areas for the Viewer App (Electron).
このドキュメントは、Viewerアプリ (Electron) のUIコンポーネント構造、視覚的レイアウト、および操作エリアを定義します。

---

## 1. Overview & Theme
## 1. 概要とテーマ

- **Theme / テーマ**: Deep Dark (Background `#0f0f12`) - Designed for immersive photo viewing. 写真閲覧に没入できるよう設計。
- **Font / フォント**: System UI Fonts (Segoe UI, Inter, San Francisco).
- **Layout / レイアウト**: 2-Pane Architecture (Main Canvas Area + Bottom Control Panel). 2ペイン構成 (メインキャンバス + 下部コントロールパネル).

---

## 2. Layout Structure (DOM Hierarchy)
## 2. レイアウト構造 (DOM階層)

### 2.1 Main Content Area (`#main-content`)
### 2.1 メインコンテンツエリア (`#main-content`)

This area is primarily dedicated to rendering the image canvas, but also hosts contextual and state-based overlays.
このエリアは主に画像のレンダリングに使用されますが、状態に応じたオーバーレイ要素も配置されます。

- **Status Bar (`#status-bar`)**:
  - Located securely at the top-left area. displays short temporal messages (e.g., "Waiting for Lightroom...", "Loading...").
  - 左上に配置。一時的な状態メッセージ（「Lightroomからの同期待ち…」など）を表示。
- **Loading Overlay (`#loading-overlay`)**:
  - Centered spinner (`animation: spin`) with "Initializing..." text indicating image processing.
  - 中央のスピナーと「Initializing...」テキストで画像処理中であることを明示。
- **Error Message (`#error-message`)**:
  - Displays a red icon and error description on image load failure.
  - 読み込み失敗時に赤いアイコンとエラー内容を中央に表示。
- **Backend Connection Lost (`#backend-dead-overlay`)**:
  - A fatal-state overlay displaying when the Python backend is disconnected. Contains a `#btn-restart-backend` button.
  - Pythonバックエンドとの接続が切れた際に画面を覆うエラー画面。「Restart Backend」ボタンを配置。
- **Main Canvas (`#main-canvas`)**:
  - The WebGL/2d context canvas for rendering the photo, focus points, and face overlays.
  - 写真、フォーカス枠、顔認識枠を描画するキャンバス。
- **Rating Overlay (`#rating-overlay`)**:
  - Located at the **Bottom Center of the Canvas**.
  - キャンバスの下部中央に配置。Lightroomの評価情報を双方向に操作・表示するためのツールバー。
  - **Rating Buttons**: ⭐️ 1〜5, and ∅ (Clear).
  - **Color Label Buttons**: Red, Yellow, Green, Blue, Purple, and None (Clear).

---

### 2.2 Bottom Panel (`#bottom-panel`)
### 2.2 下部パネル (`#bottom-panel`)

Fixed at the bottom of the window, divided into a 3-column Flexbox layout.
ウィンドウ下部に固定された、3カラム構成のFlexboxレイアウトパネルです。

#### Column 1: Information Details (左: 情報パネル)
Contains image and camera metadata.
写真およびカメラのメタデータを詳細に表示します。
- **Information Section**:
  - `#meta-filename`: Target file name. (Highlighted)
  - `#meta-resolution` & `#meta-datetime`: Image dimensions and capture date.
  - `#meta-rating-text` & `#meta-label-badge`: Read-only indicators for current Lightroom Rating & Color Label bounds.
- **Camera Data Section**:
  - `#meta-camera`: Camera Model Name.
  - `#meta-lens`: Lens Name.
  - `#meta-settings`: The Exposure Badge (e.g., `1/1000s f/2.8 ISO 100`).

#### Column 2: Focus Target & Result (中央: フォーカス情報パネル)
Displays the focus algorithm results and area tracking specifics.
AFの動作結果やターゲットトラッキングの情報を表示します。
- **Header Structure (`focus-status-row`)**:
  - `#meta-afmode`: Base AF Mode (e.g., AF-C, AF-S).
- **Core Info (`focus-info-container`)**:
  - `#meta-tracking`: Subject tracking state.
  - `#meta-af-area`: AF Area Mode (Wide, Zone, Spot, etc).
- **Detail Log (`#meta-focus-details`)**:
  - Text-heavy raw extraction results, separated by a top border, presenting coordinate details and subject recognition states.

#### Column 3: Controls (右: コントロールパネル)
A structured grid/stack for user actions and visualization toggles.
ユーザーの操作ボタンと描画トグルを集約した領域です。
- **Vertical Tool Stack**:
  - `#btn-q-fast` (Speed): Uses embedded fast preview.
  - `#btn-q-full` (Full): Process massive raw data.
  - `#btn-lock-quality` (Lock Quality): Toggle switch to persist chosen quality across session photos (Shortcut: L).
  - `#btn-frame` (Frame): Toggle Focus Point bounding box (Shortcut: F).
  - `#btn-peaking` (Peaking): Toggle contrast-detection focus peaking (Shortcut: P).
- **Slider Container (`#peaking-controls`)**:
  - A persistent spacer container that reveals `#peaking-slider` natively when peaking is active, modifying highlight threshold dynamically.
- **Action Grid (`.action-grid`)**:
  - Structured as 3 prominent square buttons at the bottom edge.
  - `#btn-dump`: Show full EXIF data overlay.
  - `#btn-save`: Saves the current canvas state with metadata overlays baked in.
  - `#btn-close`: Close application.

---

## 3. Visual & Interaction Philosophy
## 3. UIの哲学と視覚的フィードバック

### Feedback Triggers (状態変化のフィードバック)
- Toggle states (Peaking, Frame, Quality Lock) must visibly alter their background and SVG stroke colors to explicitly represent active operations.
- ボタンのトグル状態 (ピーキング、枠、画質ロック) は、背景色とアイコン色によって明確にアクティブであることがユーザーにフィードバックされなければなりません。

### Degradation Transparency (機能制限の表示)
- Elements not relevant to the current rendering state (e.g., Peaking threshold slider) are kept in the DOM as hidden (`opacity: 0` or similar display adjustments) to ensure grid layouts never "jump" abruptly and distract the user.
- 状態によって不要となる要素（ピーキングのスライダー等）は完全にDOMを消すのではなく、レイアウトシフト（ガタつき）を防ぐためにプレースホルダーの形で隠されるべきです。

---
**Note**: Key Bindings and exact shortcuts have been entirely moved to `SPECIFICATION.md` as the unified source of truth.
**注意**: ショートカットキー等の定義は統合仕様書(SPECIFICATION.md)に集約されています。

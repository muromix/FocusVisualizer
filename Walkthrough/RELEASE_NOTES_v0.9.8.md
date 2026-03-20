# Focus Visualizer v0.9.8 Alpha Milestone (Stable Navigation & Slim Build) 🪟

**Windows: v0.9.8 Alpha (Portable) | *Mac: v0.9.5 Alpha (Update coming later)*

This update focuses on navigational integrity, package optimization, and system stability. The "SUPER SLIM" distribution is now finalized for Windows, providing a zero-dependency, ultra-lightweight environment.

---
*(Scroll down for Japanese / 日本語は後半にあります)*
---

## ✨ Release Highlights (v0.9.8 Alpha)

### 🚀 Distribution & Performance
* **"SUPER SLIM" Distribution**: The Windows package has been optimized to less than 200MB. It remains a zero-dependency, portable build—no Python or Node.js installation required.
* **Unified Log Management**: All system logs (Backend, Electron, and Lightroom Plugin) are now consolidated into a single `Logs/` directory at the project root for easier troubleshooting.

### 🎨 Navigational Stability (Atomic Navigation)
* **TaskId Guard**: Implemented a monotonic task sequencing system to prevent "stale" metadata from causing image flickers or regressions during rapid paging. 
* **Atomic State Updates**: Image loading and focus-point coordinate calculations are now synchronized to prevent the "instant enlargement" visual glitch.

### 🛠️ Bug Fixes & Refinements
* **Improved Rotation Accuracy**: Fixed minor orientation issues for certain portrait-oriented Sony and Canon RAW files.
* **Layout Stability**: Refined the bottom panel to prevent text wrapping and element collision on smaller window sizes.
* **Backend Startup Fix**: Resolved a startup issue in the standalone Slim build where the backend binary detection logic was inconsistent.
* **Other miscellaneous stability improvements and minor bug fixes.**

---

## 📥 Installation & Download
1. Download the latest **ZIP** archive for Windows. (Mac users: please continue using v0.9.5).
2. Follow the setup steps in **`INSTALL_JP.txt` or `INSTALL_EN.txt`** included in the package.
3. If you encounter issues, please report them via the **Issues** tab on GitHub.

---

## ⚖️ Disclaimer
This software is provided "as is", without warranty of any kind. Released under the [MIT License](../LICENSE). This is a Public Alpha build. Use at your own risk.

---

## ☕ Support the Project
Focus Visualizer is a gift to the photography community. If this tool saves you time, please consider supporting our continued development:
* **[Support via BOOTH (Angie's Lab)](https://angieslab.booth.pm/)**
* **[Buy Me a Coffee](https://buymeacoffee.com/muromix)**

---
---

# Focus Visualizer v0.9.8 Alpha (安定ナビゲーション & スリムビルド) 🪟

**Windows: v0.9.8 Alpha (ポータブル版) | *Mac: v0.9.5 Alpha (Mac版の更新は後日予定)*

今回のアップデートでは、ナビゲーションの整合性、パッケージの最適化、およびシステム全体の安定化に注力しました。Windows向けの「SUPER SLIM」配布版が完成し、依存関係ゼロの超軽量な環境を提供します。

## ✨ 今回のハイライト (v0.9.8 Alpha)

### 🚀 配布とパフォーマンス
* **「SUPER SLIM」配布版**: Windowsパッケージを200MB以下に徹底最適化。PythonやNode.jsの別途インストールは不要で、解凍してすぐに使えるポータブル仕様を維持しています。
* **一極集中ロギング**: バックエンド、Electron、Lightroomプラグインの全ログをルートディレクトリの `Logs/` フォルダへ集約。不具合発生時の調査が容易になりました。

### 🎨 ナビゲーションの安定化 (Atomic Navigation)
* **TaskIdガード**: 単調増加するタスクIDによる順位管理を導入。高速連打時に古いデータが新しい画像を上書きする「先祖返りフリッカー」を物理的に根絶しました。
* **アトミック状態更新**: 画像の読み込みと座標計算を完全に同期させ、画像が切り替わる瞬間の「一瞬の巨大化」を防ぐ描画ロジックを確立。

### 🛠️ バグ修正とブラッシュアップ
* **回転精度の向上**: 一部のSony/Canon製 RAWファイル（縦位置）における回転制御の微細な不整合を修正。
* **レイアウトの安定化**: ウィンドウサイズが小さい場合でも、テキストの折り返しや要素の重なりが発生しないようボトムパネルを調整。
* **バックエンド起動の修正**: スリム版（スタンドアロン）でバイナリ検出ロジックが不安定だった問題を解消。
* **その他、軽微なバグ修正とシステムの安定性向上。**

---

## 📥 インストール手順
1. Windows用の最新 **ZIP** アーカイブをダウンロードしてください。（Macユーザーの方は引き続き v0.9.5 をご使用ください）。
2. 同梱の **`INSTALL_JP.txt` または `INSTALL_EN.txt`** を読み、手順に従ってセットアップしてください。
3. 不具合等は GitHub の **Issues** セクションにて承っております。

---

## ⚖️ 免責事項
本ソフトウェアは「現状有姿」で提供され、いかなる保証も伴いません。**[MIT ライセンス](../LICENSE)** の下で公開されています。本バージョンは制作途中のアルファ版です。自己責任で御活用ください。

---

## ☕ 開発を応援する
Focus Visualizer は、写真コミュニティへのギフトとして公開されています。もしこのツールがあなたの助けになったら、開発継続へのサポートをお願いします！
* **[BOOTHで支援する (Angie's Lab)](https://angieslab.booth.pm/)**
* **[Buy Me a Coffee で支援する](https://buymeacoffee.com/muromix)**

---
*(c) 2026 muromix | Synergy of Human and Machine Intelligence.*

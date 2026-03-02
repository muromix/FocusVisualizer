# Focus Visualizer - 全体仕様書 (Master Specification - Japanese)

## 1. アプリケーション概要 (Overview)
Sonyのαシリーズ（a7R Vなど）で撮影されたRAW画像（ARW）から、メタデータ（フォーカス位置、モデル情報等）を抽出し、Lightroomとのリアルタイム同期を保ちながら高精度に可視化するフルスタック・ビューワー。

### 公開リポジトリ (Official Repository)
- **GitHub**: [https://github.com/muromix/FocusVisualizer](https://github.com/muromix/FocusVisualizer)
- **Release Repository**: [https://github.com/muromix/FocusVisualizer/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest)

### ライセンス (License)
- **MIT License**: 本プロジェクトはMITライセンスの下で公開されています。商用・非商用を問わず自由な利用・改変・再配布が可能ですが、著作権表示および免責事項の保持が必要です。
- **LICENSE**: 詳細はプロジェクトルートの `LICENSE.txt` を参照。

### 1.0 セッション初期化プロトコル (Session Initialization Protocol) - *最重要*
セッション開始時、AIは以下のファイルを「真実のソース」として最優先で読み込むこと：
1.  **`DevStory/index.md`**: 進捗履歴の全体像。
2.  **`DevStory/README.md`**: 開発ストーリーの理念と構造。
3.  **`./SPECIFICATION.md`**: 本ドキュメント（全体仕様）。
4.  **`DevStory/` フォルダの最新日報**: 直前の作業内容と課題。
5.  **`./METADATA_SCHEMA.md`**: **UIMS v1.0 (共通言語) の定義。**
6.  **`./BACKEND_MODULARIZATION_PROMPT.md`**: **バックエンド大改造の設計図。**
7.  **`Dev/_repomix_all_packs/`**: 全ソースコードの同期。

### 1.1 セキュアな通信基盤 (Secure IPC Foundation) - *2026/02/23 導入*
Electronの最新ベストプラクティスに基づき、レンダラープロセスの権限を最小化。
1.  **コンテキスト分離 (Context Isolation)**: レンダラー(`renderer.js`)からNode.js/Electron APIへの直接アクセスを遮断し、グローバル名前空間を汚染しない安全なブリッジを実現。
2.  **プリロード・ブリッジ (Preload Bridge)**: `preload.js` 内で信頼されたAPIのみを `window.api` として公開。
3.  **ファイルシステム制御**: レンダラーからの直接的なファイル書き込み (`fs.writeFileSync`) を廃止。メインプロセスがIPC経由で要求を受け取り、安全性を検証した上で代行実行する。
4.  **CSP (Content Security Policy)**: 内部サーバー(127.0.0.1)へのリクエストのみを許可し、外部ドメインへの予期せぬ通信をブロック。

---

## 2. システム構成 (System Architecture)
詳細な技術仕様は各ドキュメントを参照してください。

*   **[Lightroom Plugin (Lua)](../1.Plugin_for_Lightroom)**: 写真選択イベントの監視と通信。
*   **[Python Backend](../3.Python_Backend)**: **[詳細仕様](./BACKEND_PYTHON_SPECIFICATION.md)** - Pydantic v2 & UIMS v1.0 に基づく型安全な解析、ExifTool管理、データ配信。
*   **[Focus Point Viewer (UI)](../2.Viewer_App)**: **[詳細仕様](./UI_SPECIFICATION.md)** - プレビュー表示、AF枠描画、ユーザー操作。

**プラットフォーム**: Windows / macOS
- **バックエンド**: Python (Flask) - モジュール化サービス設計 (DI)
- **フロントエンド**: Electron (高速Canvasレンダリング)
- **通信**: HTTP + SSE (127.0.0.1に限定)

### 🔐 セキュリティとプライバシー
- **ローカル完結設計**: 本アプリケーションはオフラインでの使用を前提としています。外部サーバーとの通信は一切行いません。
- **ホスト限定通信**: すべての内部通信（Lightroomとビューアー間）は `127.0.0.1` (localhost) に限定されています。
- **データ収集なし**: 画像データ、閲覧履歴、個人情報などが収集・送信されることはありません。
- **ライセンス**: 本プロジェクトは **MIT ライセンス** の下で公開されています。

### 2.1 対応プラットフォーム (Supported Platforms)
| OS | Backend | ExifTool | 起動方法 | ステータス |
| :--- | :--- | :--- | :--- | :--- |
| **Windows 10/11** | **Portable Python (同梱)** | Vendor同梱 | **`Launcher.vbs` (ステルス)** | ✅ **産業級 (v0.9.5_Alpha)** |
| **macOS** | **focus_backend (onedir)** | **システム最優先** | **Focus Visualizer.app** | ✅ **完全対応 (v0.9.5_Alpha)** |

---

## 3. 主要機能 (Main Features)

### 📸 フォーカス可視化 & 解析
*   **高速表示**: 埋め込みプレビュー抽出により、RAW画像でも0.3秒以下の高速表示を実現。
*   **フォーカス可視化**: 緑色のAF枠、トラッキング状態（α7R V対応）、主要認識対象（瞳・顔・ターゲット）を表示。
*   **フォーカスピーキング**: 合焦部をドット表示。画質モードに連動して感度を自動調整。50msの超低遅延解析。
*   **シャッタースピード精密表示**: 実効露出時間ではなく、カメラ側の設定値（例：1/40）を優先的に表示。
*   **RAW品質表示 (File Quality)**: 保存形式、サイズ等をExifから解析して表示。
*   **日本語パス完全対応 (Mojibake Rescue)**: Windows環境の文字コード齟齬を自動修復し、日本語フォルダ内でも完璧に動作。

### 🔄 Lightroom 同期
*   **双方向ナビゲーション**: ビューアーでの写真送りがLightroom側に即時反映。
*   **評価同期**: レーティング、カラーラベル、およびフラグを即座にLightroomへ反映。

### 💎 画質制御 (Quality Mode)
| モード | 解像度 | 特徴 | 挙動 |
| :--- | :--- | :--- | :--- |
| **Speed** | 1600px | **爆速・シンプル** | **初期設定**。40msスロットルによる操作感。 |
| **Full** | オリジナル | 究極のピント確認 | 切り替え時に**自動でフォーカスポイントへ等倍ズーム**。 |

---

## 4. 操作・ショートカットキー (Keyboard & Mouse Interaction)

### 4.1 キーボード操作 (Keyboard Shortcuts)
| キー | 分類 | アクション |
| :--- | :--- | :--- |
| `Space` / `Shift` | ズーム | **精密ターゲット・ズーム** (100% ↔ Fit) |
| `←` / `→` | ナビ | 写真送り (前 / 次) |
| `Q` | 画質 | 画質モード切替 (Speed ↔ Full) |
| `L` | 画質 | 画質ロック |
| `P` | ツール | ピーキング ON/OFF |
| `+` / `-` / `↑` / `↓` | ツール | ピーキング感度 調整 |
| `F` | 表示 | フォーカス枠 表示切替 |
| `I` | 表示 | レイアウト切替 (Wide ➔ Mid ➔ Compact) |
| `A` / `X` / `U` | 同期 | フラグ設定（採用 / 却下 / 解除） |
| `1` - `5` | 同期 | レーティング設定 |
| `6` - `9` / `0` | 同期 | カラーラベル設定 |

---

## 5. 技術仕様・通信 (Technical details)
*   **HTTP (Port 8765〜)**: コマンド・データ通信。
*   **SSE `/stream` (HTTP)**: バックエンドからフロントエンドへのリアルタイムPush通知。

---

## 6. 座標変換仕様 (Coordinate Transformation)
1.  **Raw座標 (1000ベース)**: 0-1000 の正規化座標。
2.  **Preview座標**: 画像ファイルの物理サイズ（Orientation適用済み）へのマッピング。
3.  **Canvas座標**: 表示サイズ（Fit / Zoom）への最終変換。

---

## 7. UI/UX デザイン仕様 (Design Specification)
モバイル環境でも快適に使用できる「サイドビュー」を追求。
- **Intelligent Auto-Resize**: [Wide / Mid / Compact] の3段階レイアウト。
- **配置**: メインスクリーンの右端に吸着。

---

## 8. 配布・同期プロトコル (Deployment & Sync Protocol)
開発環境 (`Dev/`) から配布環境 (`dist/`) へ成果物を同期する際の管理。

---

## 9. 更新履歴 (Changelog)
- **2026/03/02**: **「オープンソース移行と極限のビルド最適化」** (MITライセンス移行、ビルドサイズ削減)
- **2026/03/01**: **「v0.9.5 Alpha：摩擦ゼロ・リリースへの完成」** (ポータブル環境、VBSランチャー)

---

**Created by muromix with Lead Engineer & Angie**
「コードは、未来の自分たちへの究極のラブレターよ。」

# Focus Visualizer - 全体仕様書 (Master Specification)

## 1. アプリケーション概要 (Overview)
Sonyのαシリーズ（a7R Vなど）で撮影されたRAW画像（ARW）から、メタデータ（フォーカス位置、モデル情報等）を抽出し、Lightroomとのリアルタイム同期を保ちながら高精度に可視化するフルスタック・ビューワー。

### 公開リポジトリ (Official Repository)
- **GitHub**: [https://github.com/muromix/FocusVisualizer](https://github.com/muromix/FocusVisualizer)
- **Release Repository**: [https://github.com/muromix/FocusVisualizer/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest)

### ライセンス (License)
- **MIT License**: 本プロジェクトはMITライセンスの下で公開されています。商用・非商用を問わず自由な利用・改変・再配布が可能ですが、著作権表示および免責事項の保持が必要です。
- **LICENSE**: 詳細はプロジェクトルートの `LICENSE.txt` を参照。

### 1.0 セッション初期化プロトコル (Session Initialization Protocol) - *CRITICAL*
セッション開始時、AIは以下のファイルを「真実のソース」として最優先で読み込むこと：
1.  **`DevStory/index.md`**: 進捗履歴の全体像。
2.  **`DevStory/README.md`**: 開発ストーリーの理念と構造。
3.  **`Dev/Walkthrough/SPECIFICATION.md`**: 本ドキュメント（全体仕様）。
4.  **`DevStory/` フォルダの最新日報**: 直前の作業内容と課題。
5.  **`Dev/Walkthrough/METADATA_SCHEMA.md`**: **UIMS v1.0 (共通言語) の定義。**
6.  **`Dev/Walkthrough/BACKEND_MODULARIZATION_PROMPT.md`**: **バックエンド大改造の設計図。**
7.  **`Dev/_repomix_all_packs/`**: 全ソースコードの同期。

### 1.1 セキュアな通信基盤 (Secure IPC Foundation) - *2026/02/23 導入*
Electronの最新ベストプラクティスに基づき、レンダラープロセスの権限を最小化。
1.  **コンテキスト分離 (Context Isolation)**: レンダラー(`renderer.js`)からNode.js/Electron APIへの直接アクセスを遮断し、グローバル名前空間を汚染しない安全なブリッジを実現。
2.  **プリロード・ブリッジ (Preload Bridge)**: `preload.js` 内で信頼されたAPIのみを `window.api` として公開。
    - `window.api.send / on / invoke`: 許可されたIPCチャンネルのみを仲介。
    - `window.api.path`: システムパス操作をメインプロセス側で安全に実行。
3.  **ファイルシステム制御**: レンダラーからの直接的なファイル書き込み (`fs.writeFileSync`) を廃止。メインプロセスがIPC経由で要求を受け取り、安全性を検証した上で代行実行する。
4.  **CSP (Content Security Policy)**: 内部サーバー(127.0.0.1)へのリクエストのみを許可し、外部ドメインへの予期せぬ通信をブロック。

## 2. システム構成 (System Architecture)
詳細な技術仕様は各ドキュメントを参照してください。

*   **[Lightroom Plugin (Lua)](../../1.Plugin_for_Lightroom)**: 写真選択イベントの監視と通信。
*   **[Python Backend](../../3.Python_Backend)**: **[詳細仕様](./BACKEND_PYTHON_SPECIFICATION.md)** - Pydantic v2 & UIMS v1.0 に基づく型安全な解析、ExifTool管理、データ配信。
*   **[Focus Point Viewer (UI)](../../2.Viewer_App)**: **[詳細仕様](./UI_SPECIFICATION.md)** - プレビュー表示、AF枠描画、ユーザー操作。

**Platform**: Windows / macOS
- **Backend**: Python (Flask) - Modular Service Architecture (DI)
- **Frontend**: Electron (High-speed Canvas rendering)
- **Communication**: HTTP + SSE (Strictly bound to 127.0.0.1)

### 🔐 Security & Privacy
### セキュリティとプライバシー
- **Offline by Design**: This application is strictly for local use. It does **NOT** communicate with any external servers.
- **ローカル完結設計**: 本アプリケーションはオフラインでの使用を前提としています。外部サーバーとの通信は一切行いません。
- **Host-Only Binding**: All internal communication (between Lightroom and the Viewer) is restricted to `127.0.0.1` (localhost). This prevents any access from other machines on your network.
- **ホスト限定通信**: すべての内部通信（Lightroomとビューアー間）は `127.0.0.1` (localhost) に限定されています。これにより、同じネットワーク内の他のマシンからのアクセスは物理的に遮断されます。
- **No Data Collection**: No image data, browsing history, or personal information is collected or transmitted.
- **License**: This project is licensed under the **MIT License**.
- **データ収集なし**: 画像データ、閲覧履歴、個人情報などが収集・送信されることはありません。
- **ライセンス**: 本プロジェクトは **MIT ライセンス** の下で公開されています。

### 2.1 対応プラットフォーム (Supported Platforms) - *2026/03/01 更新*
| OS | Backend | ExifTool | 起動方法 | ステータス |
| :--- | :--- | :--- | :--- | :--- |
| **Windows 10/11** | **Portable Python (同梱)** | Vendor同梱 | **`Launcher.vbs` (ステルス)** | ✅ **産業級 (v0.9.5_Alpha)** |
| **macOS** | **focus_backend (onedir)** | **システム最優先** | **Focus Visualizer.app** | ✅ **完全対応 (v0.9.5_Alpha)** |

**Mac環境の注意事項 (v0.9.5_Alpha〜)**:
- **スタンドアロン化**: バックエンドが `focus_backend` フォルダ構造（--onedir）として同梱されており、Pythonのインストール不要かつ爆速で起動。
- **配布形式 (Unified Folder)**: アプリ本体と、プラグイン・説明書をまとめたフォルダ（`Focus Visualizer (v0.9.5 Alpha)`）をひとまとめに配布。
- **インストール手順**: 配布フォルダごと「アプリケーション」フォルダへ移動することで、管理を容易化。
- **シームレス起動**: Lightroomプラグインから `open -b com.sonyfocusvisualizer.viewer` (Bundle ID) を用いて、アプリを直接スマートに呼び出し可能。
- **ExifTool解決**: システムにインストール済みの ExifTool (`/opt/homebrew/bin/exiftool` 等) を最優先で使用。同梱版のライブラリ不足問題を解消。
- **Gatekeeper対応**: 初回起動時はアプリ本体を「右クリック → 開く」で実行を許可してください。

---

## 3. 主要機能 (Main Features)

### 📸 フォーカス可視化 & 解析
*   **高速表示**: 埋め込みプレビュー抽出により、RAW画像でも0.3秒以下の高速表示を実現。
*   **フォーカス可視化**: 緑色のAF枠、トラッキング状態（α7R V対応）、主要認識対象（瞳・顔・ターゲット）を表示。
*   **フォーカスピーキング**: 合焦部をドット表示。画質モードに連動して感度を自動調整。50msの超低遅延解析。
*   **シャッタースピード精密表示**: 実効露出時間ではなく、カメラ側の設定値（例：1/40）を優先的に表示。
*   **RAW品質表示 (File Quality)**: ロスレスRAW、MサイズRAWなどの保存形式をExifから解析して表示。
*   **日本語パス完全対応 (Mojibake Rescue)**: Windows環境の文字コード齟齬を自動修復し、日本語フォルダ内でも完璧に動作。
*   **[鮮明度解析 (予定)]**: **[詳細計画](./PROJECT_PLAN_SHARPNESS.md)** - AF枠内の解像度スコア化。

### 🔄 Lightroom 同期
*   **双方向ナビゲーション**: ビューアーでの写真送りがLightroom側に即時反映。
*   **評価同期**: レーティング(0-5)、カラーラベル(6-9)、およびフラグ（Pick/Reject）を即座にLightroomへ反映。

### 💎 画質制御 (Quality Mode) - *2026/02/23 更新*
| モード | 解像度 | 特徴 | 挙動 |
| :--- | :--- | :--- | :--- |
| **Speed** | 1600px | **爆速・シンプル** | **初期設定**。40msスロットルによる指に吸い付く操作感。 |
| **Full** | オリジナル | 究極のピント確認 | 切り替え時に**自動でフォーカスポイントへ等倍ズーム**。<br>※ズーム済みの場合は現在位置を維持。 |

### 💾 セッション永続化 (Session Persistence) - *2026/02/15 新規*
*   **起動時自動復元**: アプリケーション起動時に、前回表示していた画像を自動的にロード・表示。
*   **状態保存**: 画像パス、画質モードを `session.json` に保存。
*   **シームレスな作業継続**: 再起動後も前回の続きから即座に作業を開始可能。
*   **保存タイミング**: 画像選択時、画質モード変更時に自動保存。


---

## 4. 操作・ショートカットキー (Keyboard & Mouse Interaction)
ショートカットキーはビューアー上での操作を対象としています。

### 4.1 キーボード操作 (Keyboard Shortcuts)

| キー | 分類 | アクション | 備考 |
| :--- | :--- | :--- | :--- |
| `Space` | ズーム | **精密ターゲット・ズーム** | 等倍(100%) ↔ 全体(Fit)の切替。合焦位置を画面中央に自動配置。 |
| `Shift` | ズーム | **精密ターゲット・ズーム** | Spaceキーと同様の挙動。LRC標準のワークフローを継承。 |
| `←` / `→` | ナビ | 写真送り (前 / 次) | Lightroomの選択も即時連動 (40msスロットル) |
| `Q` | 画質 | 画質モード切替 | Speed (高速表示) ↔ Full (高解像度表示) のトグル |
| `L` | 画質 | 画質ロック | 画質モードを固定し、写真切り替え時の自動リセットを無効化 |
| `P` | ツール | **ピーキング** ON/OFF | 合焦箇所の可視化。画質モードに合わせて感度を自動調整。 |
| `+` / `-` / `↑` / `↓` | ツール | ピーキング感度 調整 | キーボードの上下キーまたは +/- でしきい値を微調整 |
| `F` | 表示 | フォーカス枠 表示切替 | 緑色のAF枠・瞳認識枠の表示/非表示 |
| `I` | 表示 | レイアウト切替 | [Wide ➔ Mid ➔ Compact] の3段階をシームレスに循環 |
| `A` | 同期 | **採用フラグ (Pick)** | 採用 ↔ なしのトグル |
| `X` | 同期 | **却下フラグ (Reject)** | 却下 ↔ なしのトグル |
| `U` | 同期 | **未設定フラグ (Unflag)** | フラグ（採用/却下）の解除 |
| `1` - `5` | 同期 | レーティング設定 | 星 1-5。同じ数字を入力すると解除 |
| `6` - `9` | 同期 | カラーラベル設定 | 赤/黄/緑/青のラベル。同じ数字で解除 |
| `0` | 同期 | **紫ラベル** | LRC標準の 0 キーに対応。同じ数字で解除 |
| `Backspace`| 同期 | ラベル解除 | 全てのカラーラベルを消去。 |
| `S` | ツール | 画像の保存 | AF枠・メタデータを焼き込んだエクスポート用JPEGを生成 |
| `F12` | 開発 | DevTools起動 | デバッグ・ログ確認用。 |
| `Esc` | アプリ | ウィンドウを最小化 | タスクバーに縮小。 |
| (X) 閉じる | アプリ | トレイへ隠す | タスクバーから消えトレイに常駐。トレイアイコンから復旧可能。 |

### 4.2 マウス・トラックパッド操作 (Mouse & Touchpad)

| 操作 | アクション | 備考 |
| :--- | :--- | :--- |
| **ホイール (スクロール)** | シームレス・ズーム | カーソル位置を中心に直感的にズームイン・アウト。 |
| **ドラッグ** | パン (移動) | ズーム時の表示位置変更。 |
| **ダブルクリック** | **全体表示 (Fit) リセット** | どんなズーム状態からでも即座に全体表示（ホーム）へ戻る。 |

---

## 5. 技術仕様・通信 (Technical details)
*   **HTTP (Port 8765〜)**: コマンド・データ通信。ポート競合時は 8765 から順に空きを自動探索。
*   **SSE `/stream` (HTTP)**: バックエンドからフロントエンドへのリアルタイム通知（Push）。*2026/02/28 UDP廃止・SSE直結に移行*
    - `type: "update"` → 画像データの更新をビューアーへ通知
    - `type: "show"` → ビューアーウィンドウの前面表示を要求
    - `EventSource` により Chrome レンダラーが Flask へ直接購読。Electron IPC を中継しない低レイテンシ設計。
*   **診断・ログ**: **[ログ出力箇所一覧](BACKEND_PYTHON_SPECIFICATION.md#diagnostics-logging)** を参照。

### 5.1 Focus Visualizer V2 (同期アーキテクチャ) - *2026/02/17 新規*
ViewerとLightroom間の超高速・高信頼性同期を実現するアーキテクチャです。

1.  **Viewer (Renderer)**: 50msのデバウンスで、ユーザー操作を即座にバックエンドへ送信。
2.  **Backend (Python)**: アクションを受信しキューに保存。画像処理（RAW現像など）は別スレッドで実行し、LightroomへのHTTP応答は**常に0ms**で返す。
3.  **Plugin (Lua)**: 0.05秒間隔で高速ポーリング。受信したアクションをバッチ処理し、**1回のトランザクションで一括書き込み**を行う。

### 5.2 メモリ管理と安定性 (Memory & Stability) - *2026/02/21 強化*
高負荷な画像処理や長時間のセッションに耐えるための実装です。

1.  **フロントエンド画像解放 (JS Memory)**:
    - 新しい画像をロードする直前に `.onerror` / `.onload` ハンドラを `null` に設定した上で `element.src = ''` を実行する。
    - これにより、ブラウザが空のパスを解決しようとして発生させる偽の `onerror` イベントが、アプリケーションの非同期状態（`isWaitingForImage` 等）を誤って破壊するのを防ぐ。
2.  **バックエンド・ガベージコレクション (Python GC)**:
    - キャッシュ枚数を **10枚** に厳格制限。キャッシュパージ時に `gc.collect()` を強制実行し、大容量RAWデータのメモリ残存を阻止。
3.  **現像シリアル・ワーカー (Serial Preview Queue)**:
    - 重たいRawpy現像処理を、専用のシリアル・キューで管理。多重スレッドによるCPU飽和を防ぎ、連打時は「最新の写真」のみを優先現像（Pre-emption）するプロ仕様の司令塔システム。
4.  **プロセス分離とステルス起動 (Process Isolation)**:
    - **Stealth Launch (Windows)**: `Focus_Visualizer_Launcher.vbs` をエントリーポイントとし、コマンドプロンプト（黒窓）を一切表示させずにバックエンドとビューアーを同時起動。
    - **Independent Existence**: VBSラッパーや親子関係の断絶により、Lightroomが停止してもビューアーが正常に動作し続け、かつゾンビ化もしない設計。
    - **Portable Isolation**: 同梱の **Python 3.11.9 (Embeddable)** ランタイムを使用。OS環境に依存せず、常にテスト済みの安定バージョンで動作。
5.  **画質連動ピーキング感度と超速解析**:
    - モード切替時に最適な閾値を適用。さらに、表示後 50ms（FULL時）で解析を開始する極限チューニング。
6.  **環境の磐石化 (Environment Resilience) - *2026/02/27 強化***:
    - **Proactive Cleaning**: 起動時に有害な環境変数（`ELECTRON_RUN_AS_NODE`など）を自動除去し、ログ初期化のタイミングを最適化することで、起動失敗のリスクを論理的に排除。
    - **Port Follower**: サーバー起動時に動的に確保したポートを `SFV_Port.info` に書き込み、ビューワーがそれを確実に追従する「すれ違い防止」の実装。
    - **Smart Connect**: 既に健全なバックエンド（/health 応答あり）が存在する場合、二重起動を避けて既存プロセスを再利用するインテリジェンスな接続。

### 5.3 クリーン・モジュラー・バックエンド (Modern Architecture) - *2026/02/27 導入*
大規模開発に耐えうる「プロの設計」をバックエンド全面に適用しました。

1.  **UIMS v1.0 (Pydantic v2)**: 
    - `models/uims.py` にて、アプリ全体のデータ構造を厳格に定義。
    - 辞書型の「手探り」を排除し、静的型チェックと爆速なシリアライズを実現。
2.  **Dependency Injection (依存性の注入)**: 
    - `AppState` や `ImageService` を各コンポーネントに外部から注入する設計。
    - コンポーネント間の結合度を極限まで下げ、テストの容易性と拡張性を飛躍的に向上。
3.  **Bootloader の隔離 (Clean Entrypoint)**: 
    - ポート探索、パス解決、OS判定などの「インフラ層」を Bootloader へ分離。
    - `server.py` はピュアな Flask ルーティングに専念し、コードの可読性を芸術的に向上。

### 5.4 クリーン・フロントエンド・アーキテクチャ (Modern Modular Frontend) - *2026/02/28 導入*
レンダラープロセスを疎結合化し、大規模開発に耐えうる拡張性と保守性を実現しました。

1.  **Stateの一元化 (Centralized Store)**: 
    - `scripts/core/Store.js` に全ての状態（画像パス、画質モード、ズーム状態等）を「真実のソース」として集約。
    - 状態変化時に `AppEvents` を発火し、各モジュールがリアクティブに自立動作する設計。
2.  **イベント駆動アーキテクチャ (Event-Driven)**: 
    - `scripts/core/Events.js` で定義されたイベントを通じたモジュール間通信。
    - モジュール同士が互いの存在を知る必要がない疎結合な関係を維持。
3.  **Canvasの物理レイヤー分離 (Layered Rendering)**: 
    - 背面の画像描画 (`ImageLayer.js`) と前面のUI/AF枠描画 (`OverlayLayer.js`) を分離。
    - 背面の重い画像を再読み込みすることなく、透明なオーバーレイのみを 60fps で高速に再描画可能。
4.  **API通信の集約 (Service Layer)**: 
    - `scripts/services/ApiClient.js` に全通信ロジック（Fetch, Polling, Error Handling）を集約。
    - プログラム全体で統一されたエラーハンドリングと、抽象化されたデータアクセスを提供。


## 6. 座標変換仕様 (Coordinate Transformation)
AF枠を正確な位置に描画し、ズーム位置を特定するための計算基準です。

### 6.1 基本フロー
1.  **Raw座標 (1000ベース)**: SonyのExifから取得される 0-1000 の正規化座標。
2.  **Preview座標**: 読み込んだ画像ファイルの物理サイズ（Orientation適用済み）へのマッピング。
3.  **Canvas座標**: ブラウザ上の表示サイズ（Fit / Zoom）への最終変換。

### 6.2 縦向き画像 (Orientation) への対応
Exifの `Orientation` タグにより、以下の入れ替えが発生します。
- **Orientation 6/8 (縦持ち)**:
    - 描画時に `width` と `height` を入れ替える。
    - Raw座標の (X, Y) を、回転方向に応じて (Y, 1000-X) などに変換する。
- **計算順序**: 常に「回転適用後の画像サイズ」を基準にAF枠を配置し、その後にCanvasの拡大率を適用する。

---

## 7. UI/UX デザイン仕様 (Design Specification) - *2026/02/23 更新*
13インチMacBook Pro等のモバイル環境でも、Lightroomと並行して快適に使用できる「サイドビュー」を追求。

### 7.1 ウィンドウ管理 (Window Management)
- **インテリジェント・オートリサイズ (Intelligent Auto-Resize)**: 情報パネルの表示状態に応じて、ウィンドウ幅を自動的に最適化。
  - **Wide (865px)**: 全メタデータを表示し、詳細な確認を行うモード。
  - **Mid (570px)**: 通常の作業に最適な標準バランスモード（初期値）。
  - **Compact (400px)**: プレビュー表示を最小限にし、Lightroomの画面を最大限に活かすモード。
- **配置 (Placement)**: 常にメインスクリーンの**右端**に吸着して配置。
- **サイクル切替 (Cycle Toggle)**: 「i」キーまたはボタンにより、[Wide ➔ Mid ➔ Compact] の順でシームレスにレイアウトを循環切替（Hiddenは閾値以下で自動発動）。
- **マウス操作 (Mouse)**:
  - **ホイール**: マウスカーソル位置を中心に自在にズームイン・アウト。
  - **ドラッグ**: ズーム時の自由なパン（移動）。
  - **ダブルクリック**: 即座に全体表示（Fit）へリセット。

### 7.2 コントロールパネル（サイドバー）
- **垂直スタック (Vertical Stack)**: 画質モード、AF枠、ピーキング、情報表示ボタンを右端に縦一列で集約。

### 7.3 レスポンシブ情報パネル (Responsive Metadata Panel)
- **情報の自動集約 (Information Consolidation)**: 「Subject Distance（合焦距離）」を独立項目からCamera Dataグリッド内へ統合。
- **レイアウトの安定化 (Anti-Jitter)**: 評価情報の有無に左右されない固定レイアウトにより、画像切り替え時のUIの揺れをゼロに。

---

## 8. 配布・同期プロトコル (Deployment & Sync Protocol) - *2026/02/23 制定*
開発環境 (`Dev/`) から配布環境 (`dist/`) へ成果物を同期する際、以下の除外ルールを厳格に適用する。

### 8.1 必須除外リスト (Mandatory Exclusions)
同期時、以下のディレクトリおよびファイルは配布パッケージに含めてはならない。

1.  **開発用バックアップ / 一時ファイル**:
    - `Backups/` フォルダ（全階層）
    - `*.bak`, `*.old` ファイル
2.  **実行環境・キャッシュ**:
    - `__pycache__/` (Pythonキャッシュ)
    - `node_modules/.cache` (Node.jsキャッシュ)
    - `session.json` (ユーザー個別のセッションデータ)
3.  **ログファイル**:
    - `*.log`
    - `Electron_Log.txt`, `BackEnd_Log.txt`, `SFV_Debug_Log.txt` 等
4.  **AI/開発支援ツール関連**:
    - `.repomixignore`, `repomix.config.json`
    - `*-repomix.xml` (Repomix出力ファイル)
5.  **内部開発用ドキュメント**:
    - 解析用メモ、内部向け詳細仕様などの `.md` ファイル
    - (例: `# EXIF情報（わかりやすい日本語名付き）.md`)

### 8.2 同期の手順 (Sync Procedure)
- 配布用フォルダ名は `Focus_Visualizer_vX.Y.Z_Alpha_Win` (Windows) または `Focus_Visualizer_vX.Y.Z_Alpha_Mac` (Mac) の形式とする。
- ルートには、多言語対応の `INSTALLMac_JP.txt`, `INSTALLMac_EN.txt` を配置する。
- Mac版の場合、DMG内には以下の構成を配置する：
  1.  `Focus Visualizer.app` (アプリ本体)
  2. `Focus Visualizer (v0.9.5 Alpha)` (関連ファイルフォルダ)
  3.  `/Applications` (エイリアス)

### 8.3 Mac版配布仕様 (Mac Distribution Specifics) - *2026/03/01 制定*
Mac版は、「フォルダごとアプリケーションフォルダへ」というUXを採用。

1.  **配布フォルダ構成 (Inside the folder)**:
    - `Focus_Visualizer.lrplugin`: Lightroom用プラグインバンドル。
    - `はじめにお読みください (JP).txt`: 日本語インストールガイド。
    - `README (EN).txt`: 英語インストールガイド。
    - `LICENSE.txt`: ライセンス条項。
2.  **起動ロジック**: 
    - プラグインから OS コマンド `open -b com.sonyfocusvisualizer.viewer` を発行。
    - ユーザーがアプリケーションフォルダ内のどこに配置していても、OSがアプリを特定して起動する。
3.  **利点**: 
    - 他のバージョンとの共存が容易。
    - プラグイン登録先が「アプリケーションフォルダ内」に固定されるため、誤削除や場所の紛失を防げる。

## 9. 更新履歴 (Changelog)
- **2026/03/02**: **「オープンソース移行と極限のビルド最適化 (The Open Wings)」**
  - **MITライセンス移行**: プロジェクトをMITライセンスへ転換。より自由なコラボレーションが可能に。
  - **ビルド・ダイエット (9.8GB ➔ 158MB)**: ビルド成果物の再帰的取り込み（鏡の中の鏡現象）を解消し、配布サイズを劇的に削減。
  - **Mac版起動アクセラレーション**: バックエンドを `--onedir` 形式へ移行し、起動時の解凍オーバーヘッドを完全排除。レスポンスを数秒短縮。
  - **バージョン・シンクロナイズ**: Mac版も v0.9.5 Alpha へ昇格させ、Windows版と全機能を同期。
- **2026/03/01 (Late Night)**: **「v0.9.5 Alpha：摩擦ゼロ・リリースの完成」**
  - **バージョン・カウントアップ**: ポータブル化、Canon対応、日本語パス警告を含む全修正を v0.9.5 として統合。
  - **自動パッケージング・システムの確立**: 不要ファイルのお掃除から ZIP 圧縮までを完全自動化し、ミスを排除。
  - **日本語パス完全回避ガイド**: Windows環境での multibyte path トラブルを防ぐための強力な警告をドキュメントに追加。
- **2026/03/01 (Night)**: **「産業級パッケージングへの昇華 (Stealth Edition)」**
  - **ポータブルPython環境の統合**: Python 3.11.9 ランタイムを同梱し、ユーザー側の Python インストールを不要化。
  - **VBSステルス・ランチャー**: コマンドプロンプトを隠蔽して起動する `Focus_Visualizer_Launcher.vbs` を実装。UXの「ツール感」を削ぎ落とし「プロダクト」へと洗練。
  - **インテリジェント・ブート**: Electron メインプロセスがポータブル環境を最優先で検出し、自動的にパスをスイッチするロジックを搭載。
  - **ワークスペース・クリーンアップ**: ポータブル環境を Git 監視下から外し、ソースコードの本質的な変更に集中できるよう環境を整備。
- **2026/03/01**: **「Mac配布エクスペリエンスの完成 (v0.9.4_Alpha)」**
  - **統合フォルダ配布**: アプリ本体と関連ファイルをまとめたフォルダをApplicationsへ移動する、管理性の高い配布形式を採用。
  - **シームレス起動 (Bundle ID)**: `open -b` コマンドにより、Lightroomからアプリをスマートに呼び出す連動機能を実装。
  - **DMGレイアウト最適化**: 説明書・プラグインを同梱したフォルダを一目で把握できるプロ仕様のレイアウトへ。
  - **ドキュメント刷新**: インストールガイド（日・英）を最新の配布形式に合わせて全面書き換え。
- **2026/02/28**: **「インタラクションの極致と視覚的ブラッシュアップ」**
  - **合焦位置への吸い付きズーム**: Precision Target Zoom を復旧。Space/Shiftキーでの等倍拡大時、常に合焦ポイントが画面中央に来る「吸い付き」を実現。
  - **オペレーションの統合**: Spaceキーをズームトグルに割り当て、LRC標準の操作感を実現。ダブルクリックを「全体表示(Fit)へのリセット」に固定し、迷子のない操作体系を確立。
  - **画質ロック (Quality Lock)**: `L`キーによる画質固定機能を完全復旧。写真切り替え時の自動復旧を制御可能に。
  - **視覚的バグ修正**: サポートリンク（QRコード）ホバー時のパネルはみ出しを修正。
  - **既知の気になる動きの記録**: レイアウト変更（ウィンドウリサイズ）時の「白フラッシュ」を調査。プラットフォーム（Electron/Windows）の基本特性に起因するものと特定し、過度な遅延ロジックを避けて仕様として記録。
- **2026/02/27**: **「レーシング・チューニング、UIMS 1.0、および環境の磐石化」**
  - **UIMS v1.0 & Pydantic v2**: データの「型」を Pydantic でカッチリ定義。辞書型の不安を排除し、高速かつ安全なデータ通信基盤へ刷新。
  - **バックエンド・モジュール化 (DI)**: 依存性の注入（DI）を導入。インフラ層（Bootloader）とロジック層（Services）を分離し、プロレベルの疎結合・高テスト性を実現。
  - **超高速ナビゲーション**: スロットルを 40ms / ピーキング待機を 50ms へ短縮し、プロレベルの選別スピードを実現。
  - **Exif解析の極限化**:
    - 設定シャッタースピード（1/40等）を実効露出値より優先して抽出。
    - α7R V 等の「トラッキング」モード（未知のID 22番等）の解析・マッピングを完了。
    - グループタグ (`-G1`) 対応により、タグ名の衝突を完全に防止。
  - **日本語パス完全解読**: `iso-8859-1` と `CP932` 変換を組み合わせた、文字化け救済ロジックの実装。
  - **サーバー・エントリポイントの芸術化**: 泥臭いパス解決やポート探索を Bootloader に隔離し、`server.py` をルーティングに特化したクリーンな設計に。
  - **起動安定化**: 起動タイミングと環境変数の衝突を解決し、磐石な起動プロセスを確立。
  - **UI洗練**: FULLモードの初期ズームを 80% (0.8) に調整し、画質モードをシンプルな2段階に集約。
- **2026/02/23**: 
  - **描画の高速・安定化 (Snappy回復)**: 複雑な非同期ロジックを整理し、絶対的な信頼性とキビキビとした操作感（0.1s以下の応答速度）を優先。
  - **UI情報の洗練**: 合焦距離を左パネルへ集約し、メインUIから冗長なリスト項目を排除。
  - **インテリジェントUI**: ウィンドウ幅の自動可変リサイズ機能の実装。
- **2026/02/21**: **RAW品質(File Quality)表示**の実装。レイアウトのガタつき完全防止(Anti-Jitter)対応。
- **2026/02/17**: **Sniper Sync V2**実装。**α7C高解像度対応**。
- **2026/02/15**: **Mac完全対応**。ExifToolパス自動解決、セッション永続化機能の追加。
- **2026/02/13**: 座標変換仕様の明文化。Full画質モード、Orientation完全対応。
- **2026/02/07**: 詳細メタデータ表示（Shooting Info）実装。
- **2026/02/02**: UIデザイン刷新、AF枠の座標計算精度向上。

---

**Created by muromix with Angie**
"Code is the ultimate love letter to our future selves."
「コードは、未来の自分たちへの究極のラブレターよ。」

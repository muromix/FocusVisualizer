# Focus Visualizer - ビューアーUI仕様書 (UI Specification)

**Date**: 2026-03-31 (v0.9.9 Alpha 準拠予定)

本ドキュメントは、Viewerアプリ (Electron) のUIコンポーネント構造、視覚的レイアウト、および操作エリアを定義します。

---

## 1. 概要とテーマ (Overview & Theme)

- **Theme / テーマ**: Deep Dark (Background `#0f0f12`) - 写真閲覧に没入できるよう設計。
- **Font / フォント**: System UI Fonts (Segoe UI, Inter, San Francisco).
- **Layout / レイアウト**: 可変 3ステージ構成 (Wide / Mid / Compact).

### 1.1 モード別特性 (Mode Characteristics)
- **Full Mode**: 右側に固定（モニタ端）。精密なピント確認のため、100%表示やターゲットズームを活用。
- **Lite Mode**: 自由配置可能。ミニマルな選別（Culling）に特化。
    - **Power-Up (Unlocked)**: コナミコマンド等でアンロックされた状態では、Liteモードでもレーティング、ラベル、フラグ、画質切替などのフルショートカットが利用可能（レイアウト切替 `I` を除く）。
    - **GUI Minimalism**: アンロック後も「Full Modeへの昇格」ボタンなどのGUI要素を一切表示せず、ショートカットキー操作のみを許容するストイックな設計。
    - **Performance**: 初期画質は引き続き最速の **Speed (1600px)** を優先。

---

## 2. レイアウト構造 (DOM Hierarchy)

### 2.1 メインコンテンツエリア (`#main-content`)

このエリアは主に画像のレンダリングに使用されますが、状態に応じたオーバーレイ要素も配置されます。

- **Status Bar (`#status-bar`)**: 左上に配置。一時的な状態メッセージを表示。
- **Loading Overlay (`#loading-overlay`)**: 中央のスピナーとメッセージ。
- **Error Message (`#error-message`)**: 読み込み失敗時に赤いアイコンと内容を表示。
- **Backend Connection Lost (`#backend-dead-overlay`)**: Pythonバックエンドとの接続断絶時に表示。
- **Main Canvas (`#main-canvas`)**: 写真、フォーカス枠、顔認識枠を描画。
- **Rating Overlay (`#rating-overlay`)**: キャンバスの下部中央に配置。

---

### 2.2 下部パネル (`#bottom-panel`)

ウィンドウ下部に固定された、3カラム構成のFlexboxレイアウトパネルです。

#### Column 1: 左: 情報パネル
写真およびカメラのメタデータを詳細に表示します。
- **Information Section**: 
  - ファイル名、解像度、日付。
  - Lightroomのレーティング/ラベルのバッジ。
- **Camera Data Section**: 
  - カメラ・レンズ名。
  - 露出設定 (SS, F値, ISO)。

#### Column 2: 中央: フォーカス情報パネル
AFの動作結果やターゲットトラッキングの情報を表示します。
- **Header Structure**: モード表示。
- **Core Info**: トラッキング状態、AFエリア設定。
- **Detail Log**: 座標データ等の詳細。

#### Column 3: 右: コントロールパネル
ユーザーの操作ボタンと描画トグルを集約。
- **Action Grid**: 画質モード(Speed/Full)、AF枠(Frame)、ピーキング(Peaking)のトグル。
- **System Buttons**: キャプチャ保存、EXIFダンプ、閉じる。

### 2.3 レスポンシブ・ステージ (Layout Stages)
ウィンドウ幅に応じて自動的に表示要素を最適化します。
- **Wide (> 865px)**: 全情報・操作グリッドを表示。
- **Mid (> 570px)**: 情報を整理し、主要な露出設定のみ表示。
- **Compact (< 400px)**: 最小情報表示。

### 2.4 コンテキストメニュー (Context Menu)
キャンバスエリアを右クリックすることで、以下のクイック操作が可能です。
- **Next Image / Previous Image**: Lightroom 側の選択を前後に移動。
- **Copy Path**: 現在の画像のフルパスをクリップボードにコピー。
- **Open in Explorer**: 画像の保存場所をエクスプローラーで開く。

---

## 3. インテリジェント表示ロジック (Internal UI Logic)

### 3.1 ゼロ・フリッカー描画 (Atomic State Update)
- **Atomic Rendering**: 画像の DOM Element の差し替えと、その画像に基づいたズーム倍率・パン座標の計算結果を、Store経由で「一撃（Atomic）」で適用します。これにより、画像が読み込まれてから計算が終わるまでのわずかな隙間に、巨大な画像（等倍）が一瞬だけ描画されてしまう現象を物理的に解消しました。
- **Seamless Transition**: 画像パスが切り替わる際、新しい画像のサイズが確定するまでは前の画像の表示状態を維持し、準備が整った瞬間に滑らかにアトミック更新します。

### 3.2 スマート・キャッチアップ (Smart Catch-up)
- **Queue Skipping**: ユーザーがキー入力（←/→）を連打して処理キューが溜まった場合、UI側とバックエンド側が連携し、古いリクエスト表示をスキップして「最新の要求画像」へ一気にジャンプします。これにより、超高速スクロール時でもUIがフリーズすることはありません。

### 3.3 ダイナミック・ターゲット・ズーム (Intelligent Auto-Zoom)
- **100% 描画時の自動追従**: スペースキーを押して画質がオリジナルサイズになった際、メタデータ内のAF枠や顔認識枠の中で「最も重要（Locked > Face > Eye）」と判断された座標を中心点として、最適化された倍率で自動的にパン・ズームします。
    
### 3.4 Solid Frame Navigation (v0.9.7)
### 3.4 盤石なコマ送り (v0.9.7)

- **Intelligent Mode Switching**: The system monitors user interaction speed in real-time (milliseconds).
- **インテリジェント・モードスイッチング**: システムはユーザーの操作速度をリアルタイムにミリ秒単位で監視します。
  - **Browsing Mode (Rapid Paging)**: 280ms未満の連続入力時は、画質設定に関わらず軽量な SPEED プレビューのみを表示。DNGファイル等の重いフォーマットでも現像機 (`rawpy`) をバイパスし、埋め込みJPEG抽出を優先することで、ARWと同等の「爆速めくり」性能を担保します。
  - **Confirm Mode (Single Press)**: 1枚ずつ丁寧に確認する際、FULL 画質へのアップグレードが完了するまでナビゲーション（次への移動）を一時的にブロック。画質更新時も「アトミック状態更新」により、ぼやけた画像から鮮明な画像へ、サイズ変化なく一瞬で切り替わります。
- **High-Responsiveness Input (mousedown)**: To ensure zero-latency response in drag-active regions, the system uses `mousedown` instead of `click`.
- **高応答入力 (mousedown)**: ドラッグ可能領域内でも遅延なく反応させるため、システムは `click` ではなく `mousedown` を使用してOSの干渉を回避します。
- **Backend Auto-Detection**: In addition to frontend keypresses, the backend automatically detects and follows rapid operations like holding down keys in Lightroom Classic (LrC).
- **バックエンド自動検知**: キーボード入力だけでなく、LrC（Lightroom Classic）側でのキー押しっぱなし操作なども自動的に検知・追従します。

### 3.5 統一ログ管理 (Unified Log Management)
- **Centralized Logging**: All log outputs from the Python backend, Electron main process, and Lightroom Plugin are directed to a single `Logs/` directory at the project root. This simplifies multi-process debugging and provides a persistent record for stability analysis.
- **一極集中ロギング**: Python、Electron、Lightroomプラグインの各ログ出力を、プロジェクトルートの `Logs/` フォルダへ集約。複数プロセスが独立して動く本システムのトラブルシューティングを劇的に簡略化するとともに、長期的な安定稼働の記録を提供します。

### 3.6 ディスプレイ適応型配置 (Multi-Display Adaptive Positioning)
- **Mouse-Context Initial Placement (LRC追従)**: ビューワーの初回起動時、システムのプライマリディスプレイではなく「マウスカーソルのあるディスプレイ」を優先してウィンドウを生成します。これにより、LRCを操作しているモニターのすぐ隣にビューワーがパッと現れる直感的な挙動を実現しました。
- **Display Sticky (画面固定スナップ)**: モード切替（Full ↔ Lite）やリサイズ時、現在ウィンドウが表示されているディスプレイを `screen.getDisplayMatching()` で検出し、そのディスプレイの WorkArea 内で配置を完結させます。拡張モニターで使用中に、勝手にメインモニターへウィンドウが戻ってしまう現象を根絶しました。

---
**注意**: ショートカットキー等の定義は統合仕様書(SPECIFICATION.md)に集約されています。

---

## 5. モジュールとコンポーネント構造 (Module & Component Structure)

UI（`2.Viewer_App`）のコードベースは、セキュリティ設定（Context Isolation）と関心の分離（Separation of Concerns）を厳格に守って設計されています。

### 5.1 プロセス・アーキテクチャ
- **`main.js`**: Electronのメインプロセス。ウィンドウの生成（透過・枠なし・常に最前面などの制御）、Pythonバックエンドへのプロキシリクエスト、およびシステムレベルのイベントハンドリングを担当します。
- **`preload.js`**: メインプロセスとレンダラー（UI）をつなぐ安全な架け橋（Context Bridge）。`window.api` という名前空間に、安全性が確認された関数（ファイル書き込み要求など）のみを公開します。

### 5.2 プレゼンテーション層
- **`viewer.html`**: アプリケーションの骨格（DOMツリー）。全UIコンポーネントとキャンバスをホストします。
- **`viewer-lite.html`**: Liteモード専用の軽量HTML。コンパクトなUIレイアウトに特化しています。

### 5.3 ビジネスロジック層 (`scripts/` ディレクトリ配下)
UIの複雑な描画・通信ロジックは、責務ごとに分割されたモジュール群で構成されています（※ES Modules等のネイティブモジュール連携機構に準拠）。
- **SSE接続モジュール**: バックエンドの `/stream` に接続し、0msで送られてくる JSONペイロード（Base64画像＋UIMSメタデータ）のデコードと分配を行います。
- **Canvas描画モジュール**: `Image` オブジェクトのロードから、`canvas` 上への画像の転写、およびPydanticパース済みの正規化座標 (`x, y, w, h`) に対する緑枠・顔枠の描画を担当します。
- **オートズームモジュール**: 100%表示（Spaceキー）時に、Targetの座標群から「ロック状態のAF枠」や「瞳AF座標」を優先して検索し、キャンバスの `transform` オフセットを動的に計算するコアロジックです。

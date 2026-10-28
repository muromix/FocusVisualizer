# Focus Visualizer - ビューアーUI仕様書 (UI Specification)

**対象バージョン**: v0.9.9.2 Alpha

本ドキュメントは、Viewerアプリ (Electron) のUIコンポーネント構造、視覚的レイアウト、および操作エリアを定義します。

---

## 1. 概要とテーマ (Overview & Theme)

- **Theme / テーマ**: Deep Dark (Background `#0f0f12`) - 写真閲覧に没入できるよう設計。
- **Font / フォント**: System UI Fonts (Segoe UI, Inter, San Francisco).
- **Layout / レイアウト**: 可変 3ステージ構成 (Wide / Mid / Compact).

### 1.1 モード別特性 (Mode Characteristics)
- **Full Mode**: 画面の右端に開く、情報パネル付きの窓。精密なピント確認のため、等倍表示やターゲットズームを活用。
- **Lite Mode**: 常に最前面に表示される小さな窓。ミニマルな選別（Culling）に特化。使えるキーは `←` / `→`、`Space`、`Shift + M` に絞られています。
    - **Power-Up (Unlocked)**: コナミコマンドでアンロックすると、Liteモードでもレーティング、ラベル、フラグ、画質切替などのショートカットが利用可能（レイアウト切替 `I` を除く）。
    - **GUI Minimalism**: アンロック後も「Full Modeへの昇格」ボタンなどのGUI要素は表示せず、ショートカットキーで操作します。
    - **Performance**: 初期画質は **Speed (1600px)**。

---

## 2. レイアウト構造 (DOM Hierarchy)

### 2.1 メインコンテンツエリア (`#main-content`)

このエリアは主に画像のレンダリングに使用されますが、状態に応じたオーバーレイ要素も配置されます。

- **Status Bar (`#status-bar`)**: 左上に配置。一時的な状態メッセージを表示。
- **Loading Overlay (`#loading-overlay`)**: 中央のスピナーとメッセージ。
- **Error Message (`#error-message`)**: 読み込み失敗時に赤いアイコンと内容を表示。
- **Backend Connection Lost (`#backend-dead-overlay`)**: Pythonバックエンドとの接続断絶時に表示。
- **Canvas (`#canvas-wrapper`)**: 写真を描く層 (`#image-layer`) と、AF枠・顔の枠・ピーキングを描く層 (`#overlay-layer`) の2枚重ね。
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
- **Action Grid**: 画質モード(Speed/Full)、画質ロック、AF枠(Frame)、ピーキング(Peaking)のトグル。
- **System Buttons**: EXIF の JSON 書き出し、AF枠入りの画像の保存、閉じる。

### 2.3 レスポンシブ・ステージ (Layout Stages)
ウィンドウ幅に応じて自動的に表示要素を最適化します。
- **Wide (> 865px)**: 全情報・操作グリッドを表示。
- **Mid (> 570px)**: 情報を整理し、主要な露出設定のみ表示。
- **Compact (< 400px)**: 最小情報表示。

---

## 3. インテリジェント表示ロジック (Internal UI Logic)

### 3.1 ゼロ・フリッカー描画 (Atomic State Update)
- **Atomic Rendering**: 画像の差し替えと、その画像に基づいたズーム倍率・パン座標の計算結果を、Store経由で同時に適用します。画像が読み込まれてから計算が終わるまでのわずかな隙間に、巨大な画像（等倍）が一瞬だけ描画されてしまう現象を防ぎます。
- **Seamless Transition**: 画像パスが切り替わる際、新しい画像のサイズが確定するまでは前の画像の表示状態を維持し、準備が整った時点で更新します。

### 3.2 スマート・キャッチアップ (Smart Catch-up)
- **Queue Skipping**: ユーザーがキー入力（←/→）を連打して処理が溜まった場合、古いリクエストの重い処理をスキップして「最新の要求画像」を優先します。

### 3.3 ダイナミック・ターゲット・ズーム (Intelligent Auto-Zoom)
- **ピント位置への自動追従**: 全体表示でスペースキーを押すと、表示中の画像を等倍に拡大し、AF枠の中で優先度の高いもの（ロック > 瞳 > 顔 > 追尾 > 通常の枠）の位置を画面中央に合わせます。AF枠がない場合は画像の中央を表示します。もう一度押すと全体表示に戻ります。
    
### 3.4 盤石なコマ送り (Solid Frame Navigation)

- **インテリジェント・モードスイッチング**: ユーザーの操作速度をミリ秒単位で監視します。
  - **Browsing Mode (Rapid Paging)**: 280ms未満の連続入力時は、画質設定に関わらず軽量な SPEED プレビューのみを表示。DNGファイル等の重いフォーマットでも現像 (`rawpy`) を飛ばし、埋め込みJPEGの取り出しを優先します。1.5秒操作がないと解除されます。
  - **Confirm Mode (Single Press)**: FULL 画質で画質ロック (`L`) が有効なときは、FULL 画質の表示が終わるまで次の写真への移動を待ちます（応答がない場合は約2.5秒で解除）。
- **高応答入力 (mousedown)**: ドラッグ可能領域内でも遅延なく反応させるため、ボタンは `click` ではなく `mousedown` で反応します。
- **バックエンド自動検知**: ビューアーでのキー入力だけでなく、Lightroom Classic 側でのキー押しっぱなし操作なども検知して追従します。

### 3.5 統一ログ管理 (Unified Log Management)
- **一極集中ロギング**: Python、Electron、Lightroomプラグインの各ログを、アプリのフォルダ直下の `Logs/` フォルダへ集約します。

### 3.6 ディスプレイ適応型配置 (Multi-Display Adaptive Positioning)
- **Mouse-Context Initial Placement (LRC追従)**: ビューワーの初回起動時、システムのプライマリディスプレイではなく「マウスカーソルのあるディスプレイ」に窓を開きます。
- **Display Sticky (画面固定スナップ)**: モード切替（Full ↔ Lite）やリサイズ時、現在ウィンドウが表示されているディスプレイを `screen.getDisplayMatching()` で検出し、そのディスプレイの WorkArea 内で配置し直します。

---
**注意**: ショートカットキーの一覧は全体仕様書（[日本語](./SPECIFICATION_JP.md) / [English](./SPECIFICATION_EN.md)）にまとめています。

---

## 4. モジュールとコンポーネント構造 (Module & Component Structure)

UI（`2.Viewer_App`）のコードベースは、セキュリティ設定（Context Isolation）と関心の分離（Separation of Concerns）を守って設計されています。

### 4.1 プロセス・アーキテクチャ
- **`main.js`**: Electronのメインプロセス。ウィンドウの生成、Pythonバックエンドの起動と終了、およびシステムレベルのイベントハンドリングを担当します。
- **`preload.js`**: メインプロセスとレンダラー（UI）をつなぐ安全な架け橋（Context Bridge）。`window.api` という名前空間に、許可した通信（ファイル保存の依頼など）だけを公開します。

### 4.2 プレゼンテーション層
- **`viewer.html`**: アプリケーションの骨格（DOMツリー）。全UIコンポーネントとキャンバスをホストします。
- **`viewer-lite.html`**: Liteモード専用の軽量HTML。コンパクトなUIレイアウトに特化しています。

### 4.3 ビジネスロジック層 (`scripts/` ディレクトリ配下)
UIの描画・通信ロジックは、責務ごとに分割したスクリプト群で構成されています。
- **SSE接続**: バックエンドの `/stream` に接続し、送られてくる JSON（UIMSメタデータ・AF座標）を受け取ります。画像本体は、バックエンド（127.0.0.1）からURLで読み込みます。
- **Canvas描画**: 画像のキャンバスへの描画と、正規化座標 (`x, y, w, h`) に基づくAF枠・顔の枠・ピーキングの描画を担当します。
- **入力とズーム**: キーボード・マウス操作の処理と、Spaceキーでのターゲット・ズーム（3.3）の計算を担当します。

# Sniper Sync - Python Backend Specification (v2.3)
**Date**: 2026-02-02

## 1. 概要 (Overview)
このドキュメントは、アプリケーションのバックエンドロジックを担当するPythonサーバーの仕様を定義します。
従来のNode.jsベースのバックエンドを代替し、より堅牢な解析と将来的な拡張（画像解析処理など）を容易にするために設計されています。

- **役割**: Lightroom Classic (Lua) と Viewer App (Electron) の間の仲介、およびRAWファイルの解析。
- **技術スタック**: Python 3.x, Flask, rawpy, ExifTool
- **ポート**: `8765` (Localhost)

## 2. アーキテクチャ (Architecture)

### 2.1 コンポーネント構成
```mermaid
graph LR
    LRC[Lightroom Plugin (Lua)] -- POST (Polling) --> PY[Python Server (Flask)]
    PY -- JSON/Image --> EL[Electron Viewer]
    EL -- POST Action --> PY
    PY -- ExifTool/Rawpy --> RAW[RAW Files]
```

### 2.2 ファイル構成 (`3.Python_Backend/`)
- **`server.py`**: Flaskサーバーのエントリーポイント。リクエストハンドリング、キャッシュ管理、ポーリング応答を行う。
- **`server.py`**: Flaskサーバーのエントリーポイント。リクエストハンドリング、キャッシュ管理、ポーリング応答を行う。
- **`sony_exif_parser.py`**: `SonyExifParser` クラス (Updated from `focus_parser.py`)。ExifToolをラッパー経由で操作し、高度な解析を行う。
- **`exiftool_wrapper.py`**: `ExifToolWrapper` クラス。`-stay_open` モードによる常駐プロセス管理を提供し、コマンド実行ごとのオーバーヘッドを排除。
- **`requirements.txt`**: 依存ライブラリ (`flask`, `rawpy` 等)。

## 3. API仕様 (API Endpoints)

### 3.1 Lightroom Classic 連携
#### `POST /`
LightroomのLuaプラグインから定期的に（またはイベント発生時に）呼び出されるエンドポイント。
- **Request (JSON)**:
    ```json
    {
        "event": "photo_selected",
        "data": { "path": "C:\\Photos\\DSC01234.ARW" }
    }
    ```
- **Response (JSON)**:
    Viewer側で発生したアクション（評価変更、ナビゲーション要求など）を返す。
    ```json
    {
        "status": "ok",
        "actions": [
            { "type": "set-rating", "value": 5 },
            { "type": "navigate", "value": "next" }
        ]
    }
    ```

### 3.2 Viewer App 連携
#### `GET /viewer/status`
現在ロードされている画像のパスと処理時刻を返す。Electron側はこのAPIをポーリングし、画像の変更を検知する。
- **Response**:
    ```json
    { "path": "...", "timestamp": 1700000000.123 }
    ```

#### `GET /viewer/data`
解析済みのメタデータとAF枠情報を返す。
- **Response**: `focus_parser.py` の `parse()` 結果 (JSON)。
    - `metadata`: 基本Exif情報 (ISO, SS, Lens...)
    - `focusPoints`: 正規化されたAF枠座標リスト `[{x, y, w, h, type, color}, ...]`
    - `rawTags`: 生のExifデータ

#### `GET /viewer/preview`
抽出されたプレビュー画像（JPEG）をバイナリストリームとして返す。
- **Content-Type**: `image/jpeg`
- **Query Param**: `?ts=...` (キャッシュバスティング用)

#### `POST /viewer/action`
Viewer上でのユーザー操作（キー入力など）を受け取り、Lightroomへの送信キュー(`action_queue`)に追加する。
- **Request**:
    ```json
    { "type": "navigate", "value": "previous" }
    ```

## 4. ロジック詳細 (Logic Details)

### 4.1 並列解析と2段階デリバリー (Parallel Analysis & 2-Step Delivery)
最新の `process_image_worker` では、解析速度を最大化するために以下の並列設計を採用している。
In the latest `process_image_worker`, the following parallel design is adopted to maximize analysis speed.

1.  **並列スレッド起動 (Parallel Threads)**:
    - メタデータ解析 (Meta Parsing) とプレビュー抽出 (Preview Extraction) を同時に開始。
    Starts metadata analysis and preview extraction simultaneously.
2.  **順次通知 (Sequential Notification)**:
    - 枠情報が完了次第、一度目のUDP通知を送信。Viewerは即座にAF枠を表示。
    Sends the first UDP notification as soon as focal points are ready. The Viewer displays AF frames immediately.
    - 画像抽出が完了次第、二度目のUDP通知を送信。Viewerが本物の画像を表示。
    Sends the second UDP notification upon image extraction completion. The Viewer then displays the actual image.
3.  **タスクID管理 (Task ID Control)**:
    - グローバルな `current_task_id` を使用。新しい写真が選ばれると古いスレッドは自爆（Abort）し、リソースの競合を防ぐ。
    Uses global `current_task_id`. When a new photo is selected, older threads abort to prevent resource contention.

### 4.2 画像レンダリングの安定化 (Image Rendering Stability)
- **透明ダミー画像 (Transparent Placeholder)**:
    - 画像抽出中にViewerがリクエストを送った場合、エラー(404)ではなく、1x1ピクセルの透明JPEG (`EMPTY_JPG`) を返す。
    When the Viewer requests an image during extraction, the server returns a 1x1 transparent JPEG (`EMPTY_JPG`) instead of a 404 error.
    - これにより、UI上での「Failed to load」というエラー表示のちらつきを完全に解消。
    This completely eliminates the flickering "Failed to load" error on the UI.

### 4.3 キャッシュ機構 (`image_cache`)
- ナビゲーションのレスポンスを即時にするため、直近 **10枚** 分の解析結果（メタデータ＋プレビュー）をメモリ上に保持。
Keeps the latest **10 items** in memory cache (Metadata + Preview) for instant navigation.
- **RLock (Recursive Lock)**: 再入可能ロックを採用し、スレッド間の競合や自己デッドロックを回避。
Uses `RLock` to avoid racing conditions and self-deadlocks between threads.

## 5. 通信テスト (Connectivity Test)
- **`/test`**:
    - アクセスすると強制的に UDP Beacon を全Subscriberへ発火させる。
    Forces a UDP Beacon to all subscribers when accessed.
    - Lightroom連携とは独立して、サーバーとViewer間の通信が生きているかを即座に検証可能。
    Allows instant verification of server-to-viewer communication independent of Lightroom.

<a name="diagnostics-logging"></a>
## 6. 診断とログ出力 (Diagnostics & Logging)

本システムのログは複数のレイヤーで出力されており、`Electron_Log.txt` に集約されます。

### 6.1 ログの集約構造
1.  **Python (`server.py`)**: 標準出力 (print/logger)
2.  **UI (`renderer.js`)**: `ipcRenderer.send('log')`
3.  **Main (`main.js`)**: 上記をすべてキャプチャし、`Electron_Log.txt` に書き出し。

### 6.2 主要なログ出力箇所

#### 【Backend / 通信】
- **UDP通信**: パケット受信、パースエラー、ポートバインド状態。
- **タスク管理**: `[Task N] Processing Started`, `Cache Hit!` などの処理フェーズ。
- **画質モード**: `Quality Mode Updated` リクエストの受信記録。

#### 【Viewer UI】
- **ステータスバー連動**: `Switching quality...`, `Metadata Ready`, `Analyzing...`
- **ユーザー操作**: `Next/Previous Photo`, `Rating Set`, `Peaking ON/OFF`
- **重大なエラー**: `FATAL ERROR` (window.onerror)

#### 【開発用ツール】
- **DevTools (F12)**: レンダラープロセスの詳細なコンソールログを確認可能。
- **Full Dump**: `POST /viewer/full-dump` により詳細なExif解析結果のJSONを出力。

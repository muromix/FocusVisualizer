# Focus Visualizer - バックエンドPython仕様書 (Backend Specification)

**Version**: v0.9.8 (UIMS v1.1 準拠)
**Date**: 2026-03-20

本ドキュメントは、アプリ全体のロジックを司る Python サーバーの設計・実装仕様を定義します。

---

## 1. 設計思想 (Architectural Philosophy)

従来のモノリシックな設計から、**疎結合・高拡張性**なモジュラーアーキテクチャへと移行しました。

- **Service Injection**: `AppState` というコンテナを通じて状態を管理し、`ImageService` や `AnalyzerManager` などのサービスを依存性注入 (DI) パターンで各モジュールへ提供します。
- **Type Safety**: 全ての外部データ（フロントエンドへの応答など）は `pydantic` (v2) で定義された **UIMS (Universal Image Metadata Schema)** を経由し、厳格にバリデーションされます。
- **Modular Bootloader**: `server.py` はアプリケーションのエントリポイントおよび初期化（Port探索・ExifToolパス解決）に専念し、ビジネスロジックは `app/` や `analyzers/` ディレクトリに完全に分離されています。

---

## 2. フォルダ構造 (Directory Structure)

```text
3.Python_Backend/
├── server.py           # アプリケーション起動・初期化・ポート探索 
├── app/                # アプリケーションコア
│   ├── core/           # 設定値 (config.py), 状態管理 (state.py)
│   ├── models/         # データ定義 (uims.py / Pydantic)
│   ├── routes/         # Flask API ルーティング (api.py)
│   └── services/       # メイン処理 (image_service.py)
├── analyzers/          # プラグイン式 メタデータ解析エンジン
├── dist/               # (Build) スタンドアロン実行ファイル (focus_backend.exe)
└── logs/               # (Legacy/Dev) 開発ログ（配布時はルート Logs/ へ集約）
```

---

## 3. 通信プロトコル (Communication IPC)

### 📊 SSE 推論エンジン & Task Management
- **Endpoint**: `/stream` (Server-Sent Events)
- **Wait-Free Delivery**: 写真切り替えイベントを検知した瞬間、0ms で UIMS ペイロードをプッシュします。
- **Base64 Elimination (URL-based)**: 以前の Base64 埋め込み（約 1.37 倍のオーバーヘッド）を廃止。現在はプレビュー画像の **URL パス**のみを送信し、フロントエンド側で非同期にフェッチする構造に移行しました。これにより、通信帯域を節約しつつ「パラパラ漫画」級の高速レスポンスを維持しています。
- **TaskId Guard (Sequencing)**: 各リクエストに単調増加する `task_id` を付与。フロントエンド側で順位管理を行うことで、ネットワーク遅延による「表示の先祖返り」を物理的に防止します。

---

## 4. エラーハンドリングと安全性 (Robustness)

1. **DNG Performance Optimization**:
   - DNG files are optimized for rapid navigation. In 'speed' mode, heavy rawpy development is bypassed in favor of aggressive multi-tag embedded JPEG extraction (ExifTool).
2. **Mojibake Rescue**: 
   - Windows 10/11 のパス名に含まれる日本語をバイナリレベルで判定し、自動的に環境に合わせたエンコーディングに変換するロジックを搭載。
3. **Unified Logging Logic**:
   - Python 側のすべてのログ出力（Flask, ExifTool, rawpy）は、プロジェクトルートの **`Logs/`** フォルダ（配布時）または `3.Python_Backend/logs/`（開発時）に集約されます。
4. **Standalone Portability (Slim Build)**:
   - PyInstaller による `--onefile` ビルドに対応。ランタイム依存関係（DLL等）を内包し、外部 Python 環境なしでの動作を保証します。

---

**Lead Engineer & Angie**
*Precision in logic. Grace in execution.*

# Focus Visualizer: Universal Image Metadata Schema (UIMS) v1.0
**Created by muromix with Angie**
# 統合イメージメタ・スキーマ v1.0

## 📝 Overview / 概要
This schema defines the standard data structure passed from the Backend (Python/AI) to the Frontend (Viewer). Ensuring consistency across different camera models and future analysis tools.
このスキーマは、バックエンド（Python/AI）からフロントエンド（ビューアー）へ渡される標準データ構造を定義します。異なるカメラモデルや将来の解析ツール間での整合性を保証します。

---

## 🏗️ Data Structure / データ構造

### 1. Root Object / ルートオブジェクト
```json
{
  "path": "string",              // [MUST] Absolute path / 絶対パス
  "preview": "string|null",      // [MUST] Preview URL with timestamp / プレビューURL (要ts)
  "preview_b64": "string|null",  // [SHOULD] Base64 image data for instant display / 即時表示用Base64
  "is_preview_ready": "boolean", // [MUST] Extraction complete flag / プレビュー準備完了
  "quality_mode": "string",      // [MUST] 'speed' | 'full' / 現在のバックエンド画質モード
  "meta_status": "number",       // [MUST] Data completeness stage: 0=Skeletal(LRC), 1=Partial, 2=Complete(ExifTool)
  "error": "string|null",        // Error message / エラーがあれば記載
  "task_id": "number",           // [MUST] Unique ID per request for staleness guard / リクエスト順位管理ID
  "metadata": { ... },           // [MUST] Image details (PascalCase) / 画像詳細（大文字始まり）
  "analysis": { ... },           // [MUST] Analytic Results / 解析結果
  "rawTags": {}                  // [SHOULD] Raw ExifTool tag dump (camelCase key) / ExifTool生タグ（camelCaseキー）
}
```

> **Note / 注意**: `rawTags` はフロントエンドの AF 解析（metadata-parser.js）が直接参照する。
> Python フィールド名は `raw_tags` だが JSON シリアライズ時は `rawTags`（camelCase）になる。

---

### 2. `metadata` Object / メタデータオブジェクト

PascalCase 原則: 先頭は必ず大文字。

```json
{
  "FileName": "string",                 // [MUST] For UI loading detection / UI読込検知に必要
  "Make": "string",                     // [MUST] Camera manufacturer ('Sony', 'Canon', etc.) / メーカー名
  "Camera": "string",                   // Camera model name / カメラモデル名
  "Lens": "string",                     // Lens model name / レンズモデル名
  "FocalLength": "number|string|null",  // Focal length / 焦点距離
  "ISO": "number|string|null",          // ISO sensitivity / ISO感度
  "ShutterSpeed": "number|string|null", // Shutter speed / シャッタースピード
  "Aperture": "number|string|null",     // Aperture / 絞り値
  "DateTimeOriginal": "string|null",    // Shooting datetime / 撮影日時
  "Orientation": "number",             // Exif orientation (1, 3, 6, 8)
  "Rating": "number",                  // 0-5
  "Label": "string",                   // Normalized English color name ('red', 'yellow', 'green', 'blue', 'purple', '')
  "Flag": "number",                    // -1=Reject, 0=Unflagged, 1=Picked
  "AFMode": "number|string|null",      // AF mode / AFモード (Sony: AF-S/AF-C/MF)
  "Subject": "string|null",            // [NEW] Subject Recognition Result / 被写体認識結果
  "ImageStabilization": "number|string|null", // IS status / 手ブレ補正状態
  "SequenceNumber": "number|string|null",     // Burst sequence number / 連写コマ番号
  "SubjectDistance": "number|string|null",    // Subject distance (m) / 被写体距離
  "ExposureProgram": "number|string|null",    // Exposure program / 露出プログラム
  "ExposureBias": "number|string|null",       // Exposure compensation / 露出補正
  "MeteringMode": "number|string|null",       // Metering mode / 測光モード
  "SceneCaptureType": "number|string|null",   // Scene type / シーン種別
  "ReleaseMode": "number|string|null",        // Release mode / レリーズモード
  "Quality": "number|string|null"             // Image quality setting / 画質設定
}
```

---

### 3. `analysis` Object / 解析結果オブジェクト

```json
{
  "focus_points": [],  // [MUST] List of FocusPoint objects (see §4) / フォーカス枠の配列
  "faces": [],         // List of face detection dicts (unstructured) / 顔認識データ（非型付け）
  "ai_score": {
    "sharpness": 0,    // AI sharpness score (0.0-1.0) / AIシャープネススコア
    "tags": []         // AI tag list / AIタグ一覧
  }
}
```

---

### 4. `FocusPoint` Object / フォーカスポイントオブジェクト

`analysis.focus_points` の各要素。座標はすべて画像サイズに対する **正規化値 (0.0-1.0)**。

```json
{
  "type": "string",      // [MUST] 'point' | 'face_lock' | 'eye_lock' | 'animal_lock' | 'bird_lock' | 'tracking_lock'
  "x": "number",         // [MUST] Normalized X center (0.0-1.0) / 中心X座標（正規化）
  "y": "number",         // [MUST] Normalized Y center (0.0-1.0) / 中心Y座標（正規化）
  "w": "number",         // [MUST] Normalized width (0.0-1.0) / 幅（正規化）
  "h": "number",         // [MUST] Normalized height (0.0-1.0) / 高さ（正規化）
  "color": "string",     // Hex color code (default: "#4AFE16") / 枠色
  "opacity": "number",   // Opacity (0.0-1.0, default: 1.0) / 透明度
  "sizeLabel": "string", // Size label: '(S)', '(M)', '(L)', '' / サイズラベル
  "status": "string",    // 'locked' | 'tracking' | etc. / フォーカス状態
  "origin": "string"     // Data source: 'maker_note' | 'ai' / データ出所
}
```

---

## 📏 Rules of Implementation / 実装ルール
1. **Always PascalCase for Metadata**: Fields inside `metadata` MUST start with an uppercase letter to satisfy the UI's property access.
   **メタデータのPascalCase原則**: `metadata` 内部のフィールドは、UI側のアクセスに合わせて必ず大文字で始めること。
2. **Mandatory FileName**: Every response, including skeleton updates, MUST include `FileName`. If missing, the UI will fall back to "Loading...".
   **FileNameの必須化**: スケルトン応答を含む全てのレスポンスに `FileName` を含めること。欠落するとUIは "Loading..." に戻ります。
3. **Bilingual Labeling**: The `Label` field must be normalized to standard English color names on the backend (e.g., '赤' -> 'red').
   **ラベルの正規化**: 日本語版LRC等からの入力も、バックエンドで標準的な英語色名に正規化してから送信すること。
4. **rawTags key is camelCase**: The Python field `raw_tags` serializes to `rawTags` in JSON. Frontend always accesses it as `data.rawTags`.
   **rawTagsキーはcamelCase**: PythonフィールドはSnakeCase (`raw_tags`) だがJSONでは `rawTags` になる。フロントエンドは常に `data.rawTags` でアクセスすること。
5. **meta_status lifecycle**: 0 (Skeletal — LRC data only) → 1 (Partial — some ExifTool fields) → 2 (Complete — full ExifTool parse done).
   **meta_statusのライフサイクル**: 0（スケルトン：LRCデータのみ）→ 1（部分的）→ 2（完全：ExifTool解析済み）。
6. **Atomic Rendering Synchronization**: When the final UIMS payload arrives and the image is swapped, the UI MUST apply the calculated zoom/pan transformations simultaneously. This prevents the "flash of massive image" (flicker) by ensuring the browser never paints the new image with stale view coordinates.
   **アトミック表示の同期**: UIMSペイロードが到着して画像を差し替える際、UIは計算済みのズーム・パン座標を「同時（アトミック）」に適用しなければならない。これにより、ブラウザが古い座標で新しい画像を描画してしまう「一瞬の巨大化（チカつき）」を物理的に防ぐ。
7. **TaskId Guard (Staleness Protection)**: The UI MUST track the latest `task_id` received from the Backend. If a payload arrives with a `task_id` smaller than the last processed one, it MUST be discarded to prevent "rollback flickering" during rapid navigation.
   **TaskId ガード（先祖返り防止）**: UIは、バックエンドから受信した最新の `task_id` を常に記憶しなければならない。以前に受信したIDよりも小さい `task_id` を持つペイロードが到着した場合、高速ナビゲーション中の遅延したデータと判断し、表示を更新せずに破棄すること。これにより、表示が前の画像に一瞬戻る「先祖返り」を防止する。

---

## 🎨 Advantages / メリット
1. **Consistency**: The Viewer doesn't care if it's a Sony A7RV or A7C; the data format is the same.
   **一貫性**: ビューアーはカメラがα7R Vかα7Cかを気にする必要がありません。データ形式は常に一定です。
2. **Extensibility**: Adding new AI features (like Eye-AF detection) only requires adding a field to the `analysis` object.
   **拡張性**: 新しいAI機能（瞳AF検出など）を追加する際も、`analysis` オブジェクトにフィールドを追加するだけで済みます。
3. **Decoupling**: Backend can change its parsing library (ExifTool -> Native) without breaking the Frontend.
   **疎結合**: フロントエンドを壊すことなく、バックエンドの解析ライブラリ（ExifToolからネイティブ解析など）を変更できます。

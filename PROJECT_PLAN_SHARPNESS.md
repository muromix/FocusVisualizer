# Project Plan: Sharpness Analysis & Distribution (Python Backend Integration)
# プロジェクト計画: 鮮明度解析と配布（Pythonバックエンド統合版）
**Date**: 2026-02-05 (Updated)

## 1. 概要 (Overview)
**Goal**: フォーカスポイント周辺の画像の「鮮明度（Sharpness）」を解析し、合焦の確信度をスコア化して表示する。
**Approch**: スタンドアロンの `.exe` を作成する当初の計画を廃止し、稼働中の **Python Backend (`server.py`)** に解析モジュールを追加統合する。

## 2. Architecture (アーキテクチャ)

```mermaid
graph LR
    Viewer[Electron Viewer] -- 1. Request Analysis --> Server[Python Server]
    Server -- 2. Read RAW/Preview --> Lib[OpenCV / NumPy]
    Lib -- 3. Laplacian Variance --> Server
    Server -- 4. Return Score --> Viewer
```

### コンポーネント
1.  **Request**: Viewerが表示中の画像に対して解析をリクエスト (`POST /viewer/analyze_sharpness`)。
2.  **Processing**: Pythonサーバーバックグラウンドで実行。
    - 画像読み込み (`rawpy` プレビュー または OpenCV)
    - 指定座標（AF枠内）のクロップ
    - グレースケール変換
    - ラプラシアンフィルタによるエッジ検出 -> 分散値 (Variance) 算出
3.  **Response**: スコア (`0.0 - 1000.0+`) をJSONで返却。

## 3. Implementation Steps (実装ステップ)

### Phase 1: Python Module Development
- **File**: `3.Python_Backend/quality_analyzer.py` (New)
- **Functions**:
  - `calculate_sharpness(image_data, roi_rect)`: バイトデータ画像と矩形範囲を受け取り、スコアを返す。
  - 依存ライブラリ: `opencv-python-headless`, `numpy`

### Phase 2: Server Integration
- **File**: `3.Python_Backend/server.py`
- **Endpoint**: `POST /viewer/analyze_sharpness`
- **Logic**:
  - キャッシュされている画像データがあればそれを利用（高速化）。
  - AF枠座標を用いてROI (Region of Interest) を切り出し。
  - `quality_analyzer` を呼び出す。

### Phase 3: Viewer UI Update
- **File**: `2.Viewer_App/viewer.html` / `renderer.js`
- **UI**:
  - フォーカス情報パネルに「Sharpness: calculating...」→「Sharpness: 854 (High)」のように表示。
  - スコアに応じた色分け（緑=高, 黄=中, 赤=低）。

## 4. Technical Detail (技術詳細)

### Sharpness Algorithm
`Laplacian Variance` 法を採用する。
```python
import cv2
def get_sharpness_score(image):
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    score = cv2.Laplacian(gray, cv2.CV_64F).var()
    return score
```
- **メリット**: 計算が高速で、ピンボケ検知に有効。
- **注意点**: ノイズが多い高感度画像ではスコアが高く出る傾向があるため、ISO感度に応じた補正が必要になる可能性がある。

## 5. Deployment (配布)
- 既存のPython環境 (`requirements.txt`) に `opencv-python-headless` を追加するだけで対応可能。
- ユーザーに追加の `.exe` インストールを強いる必要がない。

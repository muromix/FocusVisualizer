# Sony Focus Visualizer - EXIF Metadata & Parsing Specification
**Date**: 2026-03-08 (Updated)
**Backend**: Python (Flask + ExifTool `focus_parser.py` + `app.models.uims`)

## 1. 概要
本ドキュメントは、バックエンド (`focus_parser.py` および `analyzers/` プラグイン群) におけるメタデータ解析ロジック、Sonyメーカーノート特有の仕様定義ドキュメントです。
フロントエンドへのペイロード形式の詳細は、**[METADATA_SCHEMA.md](./METADATA_SCHEMA.md)** (UIMS v1.0) を参照してください。

## 2. データ配信構造 (Data Delivery)
`GET /viewer/data` のようなレガシーなポーリングは廃止され、現在は `/stream` エンドポイントを通じた **SSE Data Push** によりデータがフロントエンドへ直接注入されます。

JSON構造やデータの持ち方については、すべて Pydantic で管理される `app.models.uims.UIMSPayload` に統合されています。

## 3. 解析ロジック (Implementation Details)

### 3.1 クラス設計: `SonyFocusParser`
**役割**: RAWファイルのパスを受け取り、ExifTool を Stay-Open モードで駆動してメタデータを抽出する基盤クラス。
- **依存**: `exiftool.exe` (Stay-Open Mode)
- **連携**: 抽出された生データ (`raw_tags`) は `AnalyzerManager` に渡され、各モジュール（Sony, Canon用アナライザー等）によって UIMS フォーマットへ正規化して変換されます。

### 3.2 Sony MakerNotes 主要タグ定義

#### フォーカスモード (0x2011 / FocusMode)
| Value | Mode | Note |
| :--- | :--- | :--- |
| `0` | Manual | MF |
| `2` | AF-S | Single-shot AF |
| `3` | AF-C | Continuous AF |
| `4` | DMF | Direct Manual Focus |
| `6` | AF-A | Automatic AF |

#### フォーカスエリアモード (AFAreaMode)
文字列パースによる判定を行う（ExifToolのデコード結果を利用）。
- `Tracking: Zone` -> トラッキング有効
- `Animal Eye Tracking` -> 動物瞳AF有効
- `Wide`, `Zone`, `Center` -> 通常エリア

## 4. 座標正規化 (Normalization)
ExifToolの出力（ピクセル座標）を、Viewer用の相対座標（0.0-1.0）へ変換する計算式。
現在は `ImageService` や `rawpy` ではなく、パーサーやアナライザーで画像の元の Orientation を用いて論理的に正規化を行っています。

```python
# Pseudo Code for Coordinate Normalization
img_w, img_h = exif_width, exif_height
# Orientationによる入れ替えをあらかじめ考慮しなくても、描画側で適切に解釈される場合はそれに従う
# あるいはフロントエンド側で CSS transform を利用してアジャストする設計思想に準拠

norm_x = (raw_x - raw_w / 2) / img_w
norm_y = (raw_y - raw_h / 2) / img_h
norm_w = raw_w / img_w
norm_h = raw_h / img_h
```

## 5. ⚠️ 解析途上課題: α7R V対応 (Tag9401 Deep Dive)
- **過去の課題**: α7R V (ILCE-7RM5) などの最新世代では、`FocusLocation` が従来とは異なるバイナリ形式の `Sony:Tag9401` にカプセル化され、ExifToolでそのまま読めませんでした。
- **一時的な解決と現状**: Python側のバイナリ抽出アナライザーにより、`Tag9401` のバイト列をリバースエンジニアリングする処理を実装し、一部の顔認識や瞳AFの枠抽出には成功しました。
- **お蔵入り理由（今後の課題）**: しかし、ノイズデータ（背景のオブジェクトなどを誤検知したと思われる微小な「顔」データ）などが1回の解析で **150個以上** 抽出されてしまうオーバーディテクションの現象が確認されました。そのため、表示の混乱を避けるため、現在は機能を一時的に制限（お蔵入り）し、真に「ユーザーが合焦した／意図した被写体」だけをフィルタリングするための追加解析（自信度スコアやフラグの解明）が求められています。

## 6. デバッグと拡張性
- 未解明のメタデータや新しいカメラボディが登場した場合は、`rawTags`（元の生データ群）を参照し、`analyzers/` フォルダ配下に新しいアナライザーモジュールを追加することで解析機能を追加可能です。


# Sony a7RV (ILCE-7RM5) Subject Recognition Analysis Progress
# Sony a7RV (ILCE-7RM5) 被写体認識解析の進捗状況

## 1. Executive Summary / 概要
Through extensive comparison of RAW samples (v4.00 firmware), we have successfully mapped the nested subject recognition structure of Sony a7RV.
数多くの RAW サンプル（v4.00 ファームウェア）の比較により、Sony a7RV の入れ子構造になった被写体認識メタデータの解読に成功しました。

### Current Status / 現在のステータス
- **Implementation:** "Shallow but Robust" Spec (v4.00 optimized).
- **実装状況:** 「浅層かつ堅牢（ロバスト）」な仕様（v4.00 最適化版）。
- **Coverage:** Human (100%), Subject Tracking (100%), Vehicles/Wildlife differentiation (Validated).
- **カバー範囲:** 人物 (100%)、被写体認識 (100%)、乗り物/野生動物の切り分け（検証済み）。

---

## 2. Discovery: Subject ID Mapping / 被写体IDのマッピング結果

### Layer 1: Mode Selection (AFAreaMode) / 第一階層: 認識モード
| ID | Display Label (JP/EN) | Meaning / 意味 |
| :--- | :--- | :--- |
| **14** | **Human Priority / 人物優先** | Real-time Tracking (Human) / 人物優先トラッキング |
| **20** | **Subject Recog. / 被写体認識** | Non-human Subject Recognition / 被写体認識モード |
| **3** | **Flexible Spot / フレキシブルスポット** | Recognition: OFF (Standard AF) / 認識OFF (通常のAF) |

### Layer 2: Subject Sub-category (0x940e_0x203a) / 第二階層: 被写体サブカテゴリ
*Only applicable when Layer 1 is 20.*
*第一階層が 20 の時のみ有効。*

| Hex | Decimal | Subject Type / 被写体種別 | Reference Sample / 検証サンプル |
| :--- | :--- | :--- | :--- |
| **0x36** | **54** | **Vehicle / 乗り物** (Car, Train) | DSC00384 (Car) |
| **0x00** | **0** | **Wildlife / 野生動物** (Bird, Animal, Insect) | DSC00374, DSC00382, DSC00383 |

---

## 3. Notable Findings / 特記すべき発見

### The "Animal Eye" Mislabeling / 「アニマルアイ」の誤記問題
- **Observation:** Sony uses the label `Animal Eye Tracking` (Old ID 20) in EXIF for ALL subjects (Bird, Insect, Car, etc.).
- **発見:** α7R V は「鳥・虫・車」すべてを Exif 上で **`Animal Eye Tracking`** という古いラベルの場所に一括記録している。
- **Action:** Our Viewer overrides this to **`Subject Recognition`** to ensure accurate display for cars and insects.
- **対応:** Viewer ではこれを汎用的な **「被写体認識」** に上書きし、誤解を招く「動物瞳」という表記を修正。

### Discovery of 0x2030 (Potential Direct ID) / 0x2030 (真のID候補) の発見
During analysis, `Sony_Tag940e_0x2030` showed unique IDs:
解析中、`Sony_Tag940e_0x2030` に固有の ID が現れた:
- Human: 122
- Bird: 16 (Sample 1), 36 (Sample 2) -> Changed per file?
- Animal: 217
- Insect: 55
- Vehicle: 62
*This tag is not yet stable enough for implementation but remains a key area for future research.*
*このタグは実装に耐えるほど安定していない (Bird で変化) ため、将来の研究課題とする。*

---

## 4. Unresolved Items / 今後の課題

- **Differentiating specific Wildlife**: Bird vs Animal vs Insect separation is still dependent on Layer 1 mapping.
- **野生動物内の切り分け**: 「鳥・動物・虫」をさらに細かく分離するための安定したフラグの特定。
- **Firmware Adaptability**: Keep monitoring if Sony v5.0+ or newer models change these top-level IDs.
- **ファームウェア適応性**: v5.0+ 以降や新機種でこれらのトップレベル ID が変更されないかの監視。

---

## 5. Reference Samples Used for Analysis / 解析に使用したサンプル
- **Human (378)**: Human Tracking.
- **Bird (374, 385, 048, 070)**: 048 was older/different setting (ID 3).
- **Animal (382)**: Animal Recognition.
- **Insect (383)**: Insect Recognition.
- **Vehicle (384)**: Car/Train Recognition.
- **No Face (380)**: Landscape / No Tracking.

---
最後更新日: 2026-03-19
Last Updated: 2026-03-19

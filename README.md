# 🎯 Focus Visualizer (Alpha Releases)

> **Speed meets precision.**  
> A dedicated viewer demo to instantly check the "focus point" of Sony Alpha RAW photos selected in Adobe Lightroom Classic.
> 
> Adobe Lightroom Classic で選択した Sony αシリーズの RAW 写真から、「フォーカス位置」を爆速で確認できる専用ビューアーの配布（デモ公開）リポジトリです。

![Focus Visualizer Header](https://img.shields.io/badge/Status-Public_Alpha-orange) ![Platform](https://img.shields.io/badge/Platform-Mac_%7C_Windows-blue)

## 📥 How to Download / ダウンロード方法

All installation packages (ZIP / DMG) are hosted in the **Releases** section.
すべてのインストールパッケージ（ZIP / DMGファイル）は **Releases** ページにて配布しています。

👉 **[Go to Latest Release (最新のリリースへ)]([https://github.com/muromix/Focus-Visualizer-Alpha-Releases/releases/latest](https://github.com/muromix/FocusVisualizer/releases/latest))**

* Click the link above or check the "Releases" section on the right sidebar of this page.

---

## 🎯 What is Focus Visualizer? / ツール概要

Focus Visualizer is a high-speed workflow utility for Sony photographers using Adobe Lightroom Classic. It visualizes:
- **Exact AF Points**: Real-time localization of green AF frames.
- **Deep Metadata**: Camera data, focus distance, and RAW quality details.
- **LRC Two-Way Sync**: Ratings, Color Labels, and Flag (Pick/Reject) synchronization.

Lightroomで写真を選んだ瞬間、別ウィンドウでAF枠や瞳認識の位置を爆速で表示します。
α7R Vを筆頭とするSony最新世代のカメラから引き出せる全ての「合焦情報」の可視化と、レーティングやフラグの「双方向同期」を可能にします。

## 💻 Recommended Specs / 動作推奨環境

- **OS**: macOS (Apple Silicon) OR Windows 10 / 11 (64bit)
- **Adobe Lightroom Classic** (v12.0+)
- **System**: 8GB+ RAM, SSD highly recommended

*(※ Notice / 注意: Image analysis requires `python3` and `ExifTool`. Installation instructions are included within the downloaded ZIP or DMG file. / 画像解析のため、Python3 と ExifTool の導入が必要になります。導入手順はダウンロードしたZIPやDMG内に同梱されています。)*

---

## 🛡️ Privacy & Security / プライバシーと安全性

- **100% Offline / オフライン動作:**
  Focus Visualizer operates entirely locally. **Absolutely no data / photos are sent over the internet.** It only communicates internally within your PC (127.0.0.1).
  本ツールのすべての処理はご自身のPC内部のみで完結します。インターネット等の外部へ写真データ等を送信する処理は含まれていません。（Lightroomとビューアー間の通信は、PC内部のローカル通信のみを使用しています）

- **No Registry Modifications / クリーンな非破壊設計:**
  This application is portable and does not mess with your system registry. To uninstall, simply delete the folder.
  本ツールはWindowsのレジストリやシステムの深部等のコア設定を書き換えないポータブル設計です。アンインストールしたい場合は、ダウンロードしたフォルダを丸ごと削除するだけで消去できます。

---

## ⚠️ Known Limitations / 既知の制限事項

As this is a Public Alpha release, please be aware of the following technical limitations:
本バージョンはパブリック・アルファ版であるため、以下の技術的な制限や環境依存事項が含まれます。

- **Multi-process Configuration / マルチプロセス構成による環境への影響:**
  The app relies on Lightroom, Python, and Node.js working together locally. Strict security software or firewall settings might accidentally block the internal communication between them.
  内部通信を使用して3つのプロセスが連携動作するため、ご使用のセキュリティソフト等の設定によっては内部通信が遮断され、動作に影響が出る場合があります。
- **Resource Intensive (Full Mode) / 高負荷時のメモリ消費:**
  Rendering high-resolution RAW files in "Full" quality mode consumes significant system memory. Rendering delays may occur on lower-end machines.
  Full（最高画質）モードでのRAW現像・描画処理はPCのメモリを大きく消費します。スペックの低いPCでは描画に遅延が発生する場合があります。

---

## ⚖️ Disclaimer / 免責事項 (Please Read)

**This software is provided "as is", without warranty of any kind.**  
In no event shall the developer be liable for any claim, damages, data loss, or other liability arising from the use of this software. Please use it at your own risk.

**本ソフトウェアは「現状有姿」で提供され、いかなる保証も伴いません。**  
本ツールの使用によって生じたデータの破損、PCの不具合、その他直接的・間接的な損害等について、開発者は一切の責任を負いかねます。事前に動作確認の上、ご自身の責任（自己責任）においてご利用をお願いいたします。

---

## 🐞 Bug Reports & Feedback / 不具合報告・フィードバック

If you encounter any unexpected behavior, please let us know via the **[Issues](https://github.com/muromix/Focus-Visualizer-Alpha-Releases/issues)** tab!
ページ上部の **Issues** タブからバグ報告や改善要望をお寄せいただけると大変励みになります。

## ☕ Support the Project / 開発を応援する

Focus Visualizer is a labor of love for Sony Alpha users. If you find this tool useful, consider supporting the development!
このプロジェクトは個人の情熱で開発されています。便利だと感じていただけたら、ぜひサポートをお願いします！

[![Buy Me a Coffee](https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20coffee&emoji=☕&slug=muromix&button_colour=FF5F5F&font_colour=ffffff&font_family=Cookie&outline_colour=000000&coffee_colour=FFDD00)](https://buymeacoffee.com/muromix)

<br>

*(c) 2026 muromix | Made with passion for Sony Alpha (and beyond).*


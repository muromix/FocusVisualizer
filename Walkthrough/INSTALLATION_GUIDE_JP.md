# Focus Visualizer - インストールと設定ガイド (v0.9.8 Alpha)

**Created by muromix & Angie**

このドキュメントは、Adobe Lightroom Classic (LRC) 用「Focus Visualizer」のセットアップ手順書です。
v0.9.8 "SUPER SLIM" 版の最新ロジックを反映しています。

---

## 🚀 1. システム要件 (準備)

本バージョンは、**Windows においては Python や Node.js の事前インストールが不要** な「ポータブル版」として開発されています。

### 🪟 Windows をお使いの方
- **OS**: Windows 10 / 11 (64bit)
- **依存環境**: 不要（全てパッケージに内蔵されています）
- **Lightroom Classic**: v12.0 以上推奨

### 🍎 Mac をお使いの方
- **OS**: macOS 12.0 (Monterey) 以上推奨
- **最新状況**: 現在、Mac 版は **v0.9.5 Alpha** です（v0.9.8 相当へのアップデートは後日予定）。

---

## 📦 2. セットアップ手順 (Windows)

Windows 版は「解凍して置くだけ」で準備が完了します。

1. ダウンロードした ZIP ファイルを、任意の場所に解凍（展開）します。
2. フォルダを配置したら、Lightroom Classic を起動してください。
3. **[ファイル] ＞ [プラグインマネージャー]** を開きます。
4. **[追加]** ボタンをクリックし、同梱されている **`1.Plugin_for_Lightroom`** フォルダを選択します。
5. 緑色の丸が表示され「プラグインは有効です」となれば完了です。

---

## 🎯 3. 起動方法 (LRC から起動)

本ツールは、Lightroom Classic 内から直接メニューを呼び出すことで、ビューアーが自動的に立ち上がります。

1. Lightroom Classic のライブラリモジュール等で、写真を選択します。
2. **[ファイル] ＞ [プラグインエクストラ] ＞ [Focus Visualizer for Sony Alpha]** をクリックします。
3. ビューアーが自動的に立ち上がり、写真の合焦位置を可視化します。

---

## 🛠️ 4. メンテナンスとトラブルシューティング

- **統合ログ (`Logs/`)**: 
  バックエンド、Electron、Lightroomプラグインの全ての動作ログが、ルート直下の **`Logs/`** フォルダに出力されます。不具合が発生した際はこの中のファイルを確認してください。
- **ポータブル設計**: 
  レジストリ等のシステム設定は書き換えません。アンインストール時は、プラグインマネージャーから削除し、フォルダごとゴミ箱へ捨てるだけで安全に削除できます。

---

## ☕ 開発を応援する

Focus Visualizer は、写真家のワークフローを劇的に加速させるためのオープンソースプロジェクトです。
もしこのツールがあなたの助けになったら、開発継続へのサポートをお願いします！

- **[BOOTHで支援する (Angie's Lab)](https://angieslab.booth.pm/)**
- **[Buy Me a Coffee で支援する](https://buymeacoffee.com/muromix)**

---
*Focus Visualizer - Developed by muromix & Angie.*

# Focus Visualizer - インストールと設定ガイド

このドキュメントは、Adobe Lightroom Classic用「Focus Visualizer」のセットアップ手順書です。リリース時のReadmeテキストの生成ベースとして使用されます。

## 1. システム要件 (準備)

Focus Visualizer の動作には以下の環境が必要です。

### Windows をお使いの方
- **OS**: Windows 10 / 11 (64bit)
- **Python**: バージョン 3.10 以上 (PCにインストールされている必要があります)
- **Lightroom Classic**: v12.0以上推奨
- **メモリ**: 8GB 以上 (16GB以上推奨)

### Mac をお使いの方
- **OS**: macOS 12.0 (Monterey) 以上推奨
- **Node.js**: v18 LTS 以上推奨 ([ダウンロード](https://nodejs.org/))
- **Python**: バージョン 3.10 以上推奨 (`setup_mac.command` が自動確認します)
- **ExifTool / Homebrew**: 未導入の場合も `setup_mac.command` が自動インストールします

---

## 2. 設置と起動

### 🪟 Windows の場合

1. ダウンロードした「Focus_Visualizer」フォルダを任意の場所に配置します。
2. フォルダ直下の **`setup_win.bat`** をダブルクリックして実行します。
   （Pythonの仮想環境の構築と、必要なライブラリの自動インストールが行われます）

### 🍎 Mac の場合

1. フォルダ直下の **`setup_mac.command`** をダブルクリックして実行します。
   *（セキュリティ警告が出た場合: 右クリック → [開く] → [開く]）*
2. スクリプトは以下を自動で実行します：
   - Homebrew の確認・インストール
   - ExifTool の自動インストール
   - UI依存パッケージのインストール (`npm install`)
   - Python 仮想環境の作成とライブラリ展開
   - 実行権限の自動付与
3. 「🎉 Setup Complete!」と表示されたら準備完了です。

---

## 3. Lightroom Classic へのプラグイン登録設定

この設定はWindows・Mac共通です。

1. Lightroomを開き、**[ファイル] ＞ [プラグインマネージャー]** を選択します。
2. **[追加]** ボタンをクリックし、同梱されている `1.Plugin/Focus_Visualizer.lrplugin` フォルダを選択します。
3. 登録が完了すれば準備OKです！ 写真を選択して **[ファイル] -> [プラグインエクストラ] -> [Focus Visualizer for Sony Alpha]** を実行するだけで、全自動で起動します。

---

## 4. アンインストール

本アプリケーションはレジストリ等のシステム設定を変更しません。安全にアンインストールするには以下の手順を実行してください。

1. Lightroom Classic を開き、[プラグインマネージャー] から「Focus Visualizer for Sony Alpha」を選択し、[削除] をクリックします。
2. 配置したこのアプリケーションフォルダごと「削除（ゴミ箱へ移動）」してください。
3. Python や Node.js 自体が不要になった場合は、OSのシステム設定から手動でアンインストールしてください。

---
*Focus Visualizer - Developed by muromix & Angie.*

# Focus Visualizer - Release Protocol (SLIM Edition)
# Focus Visualizer - リリース・プロトコル (SLIM版)

---

## 1. Overview / 概要
This document defines the process for creating a "SUPER SLIM" distribution of Focus Visualizer. The goal is to keep the package under 200MB by pruning unnecessary runtimes and localized assets.
このドキュメントは、Focus Visualizerの「SUPER SLIM」配布版を作成するプロセスを定義します。不要なランタイムやローカライズ資産を削ぎ落とし、パッケージを200MB以下に抑えることが目標です。

## 2. Automated Build Script / 自動ビルドスクリプト
The master build script is located at:
マスタービルドスクリプトの場所：
`Build_Scripts/build_win_slim.ps1`

### Usage / 使い方
Run the script via PowerShell, passing the desired version number.
PowerShellから実行し、バージョン番号を指定します。

```powershell
# Example: Creating v0.9.9
powershell -ExecutionPolicy Bypass -File Build_Scripts/build_win_slim.ps1 -Ver "0.9.9"
```

## 3. Slim Standards / スリム化の基準
The automated script enforces the following pruning:
自動スクリプトにより、以下の項目が自動的に削減されます：

1.  **Electron Runtime Diet**: 
    - Removes all locales except `en-US.pak` and `ja.pak`.
    - Locales以外の不要なランタイム資産を削除。
2.  **Node.js Cleanup**:
    - Executes `npm ci --omit=dev` to ensure zero development dependencies.
    - 開発用モジュールを一切含まないクリーンなモジュール構成。
3.  **Junk Removal**:
    - Deletes `.git`, `.bak`, `.tmp`, and Mac-specific documentation in the Windows build.
    - Windows版に不要なMac用ドキュメント（INSTALLMac等）や、中間ファイルを完全に除去。
4.  **Log Unification**:
    - Ensures all logs (LRC, Backend, Electron) are unified into a single `Logs/` folder at the root.
    - すべてのログをルート直下の `Logs/` フォルダへ集約し、ポータブル性を向上。

## 4. Version Management / バージョン管理
The script automatically replaces version strings in:
スクリプトは以下のファイルのバージョン表記を自動的に置換します：
- `Focus_Visualizer.bat` (Window Title & Echo)
- `Docs/INSTALL_JP.txt`
- `Docs/INSTALL_EN.txt`
- Folder Names and Zip file naming.

## 5. Pre-Release Checklist / リリース前チェックリスト
Before zipping, the script performs these final verification steps:
圧縮前に、スクリプトは以下の最終確認を行います：
- [x] Backend binary (`focus_backend.exe`) exists.
- [x] ExifTool handles are correctly placed in `vendor/`.
- [x] Root `Logs/` directory detection is active.

---
**Maintained by Antigravity (Angie)**
"Reproducibility is the bedrock of professional engineering."
「再現性こそが、プロフェッショナルの証明よ。」

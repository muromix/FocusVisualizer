# Architecture: 4-Layer Metadata Fortress
# 設計思想：4層のメタデータ要塞 (4-Layer Metadata Fortress)

## 1. Objective / 目的
In a multi-process environment where Lightroom Classic (Lua), the Python Backend, and the Electron Frontend operate at different speeds, metadata (Stars, Labels, Flags) can become inconsistent due to race conditions. The "4-Layer Metadata Fortress" is a protective architecture designed to govern this asynchronous world and ensure that user actions are never lost or reverted.
Lightroom Classic (Lua)、Pythonバックエンド、Electronフロントエンドがそれぞれ異なる速度で動作するマルチプロセス環境において、レースコンディションによってメタデータ（星、ラベル、フラグ）が不整合になることがあります。「4層のメタデータ要塞」は、この非同期な世界を支配し、ユーザーの操作が失われたり巻き戻されたりしないことを保証するために設計された防衛アーキテクチャです。

> **"Asynchronous inconsistencies should not be waited for serially, but governed through fortification."**
> **「非同期処理のすれ違いは、直列に待つのではなく、要塞化して支配する」**

---

## 2. The Four Layers of Defense / 4つの防衛階層

### Layer 1: Heartbeat Poison Filter (Heartbeat Guard)
### 第1層：無意識の毒の無力化 (Heartbeat Guard)
- **Location**: Backend Entry (`api.py` -> `lrc_endpoint`)
- **Role**: Lightroom sends "Heartbeat" events every 100ms. These events often contain empty values (Rating: 0) because real metadata is only fetched on explicit photo changes. This layer treats these empty values as "poison" and prevents them from resetting the backend's memory for the current image.
- **場所**: バックエンド入口 (`api.py` -> `lrc_endpoint`)
- **役割**: Lightroomは100msごとに「生存確認（Heartbeat）」を送信しますが、これには空の評価値（Rating: 0等）が含まれることがよくあります。この層はそれらを「毒」と見なし、バックエンドの記憶がリセットされるのを防ぐ最初の防波堤となります。

### Layer 2: Action Memorization (Action Guard)
### 第2層：行動の即時記憶 (Action Guard)
- **Location**: Backend Action Handler (`api.py` -> `/viewer/action`)
- **Role**: When a user sets a rating (e.g., "5 stars") in the viewer, the backend immediately commits this action to its own state memory without waiting for Lightroom to write to the file. This ensures that even if ExifTool reads a "stale" file a moment later, the user's latest intent is preserved.
- **場所**: バックエンド・アクション受付 (`api.py` -> `/viewer/action`)
- **役割**: ビューワー上でユーザーが「星5」をつけた瞬間、Lightroomがファイルに書き込むのを待たずに、バックエンド自身のメモリ（state）にその行動を即座に刻み込みます。これで、ExifToolが直後に古いファイルを読み込んでも「記憶」が優先されます。

### Layer 3: Authoritative Injection (Injection Guard)
### 第3層：出口での強制注入 (Injection Guard)
- **Location**: Backend Exit (`api.py` -> `/viewer/data` & SSE Broadcast)
- **Role**: Before sending data to the frontend, the backend injects its latest "authoritative" knowledge (the LrC state stored in Layer 2) into the payload. It also elevates the `meta_status` level to 1 (Authoritative), signaling the frontend to trust this data over raw file tags.
- **場所**: バックエンド出口 (`api.py` -> `/viewer/data` ＆ SSEブロードキャスト)
- **役割**: フロントエンドへデータを発送する直前に、バックエンドが保持する「正しい記憶（LrCの最新状態）」をペイロードに、強制的に上書き注入します。また、`meta_status` を確実に引き上げ、フロントエンドに対して「このデータは信頼せよ」というシグナルを送ります。

### Layer 4: Absolute UI Defense (Authority Guard)
### 第4層：フロントエンドの絶対防衛壁 (Authority Guard)
- **Location**: Frontend Data Store (`Store.js` -> `setImageData`)
- **Role**: The final gatekeeper. Even if the backend fails and sends "empty" data (due to reloads or initialization), the frontend refuses to drop the correct metadata it already holds unless the newcomer has a higher "Authority Level." It handles type coercion (string "0" vs int 0) to prevent accidental wipes.
- **場所**: フロントエンド (`Store.js` -> `setImageData`)
- **役割**: 最後の門番です。万が一、バックエンドが空のデータや型の違うゴミを送ってきても、画面に表示されている正しいメタデータを絶対に手放しません。「権限レベル」の低い更新が、実質的な「情報の消失」を引き起こすのを防ぎます。

---

## 3. Impact / 効果
By coordinating these four layers, the user experiences a rock-solid UI where metadata never flickers or vanishes, regardless of:
- Rapid Quality Mode switching (SPEED ⇔ FULL).
- Continuous browser reloads.
- Lightroom API synchronization delays.
- File system read/write latency.
この4つの層が完璧に噛み合っているため、以下の状況下でもメタデータは微動だにせず、盤石なUI体験を提供します：
- 高速な画質モード切り替え（SPEED ⇔ FULL）。
- 連続するブラウザのリロード。
- Lightroom APIの同期遅延。
- ファイルシステムの読み書きレイテンシ。

# ZMK v0.3-branch 安定ビルド実現に向けたゼロベースエラー解析および修正計画

コミット `fcf1c9b0` の状態から発生するエラーの根本原因をゼロベースで精査し、ZMK `v0.3-branch` 環境において確実にビルドを成功させるための対応計画です。

## 原因（Build Error Root Causes）
コードベース（`fcf1c9b0`）を詳細に解析した結果、以下の **4つの問題（バージョンの不一致および構文エラー）** が特定されました：

1. **【原因1】`build.yaml` における未対応 Snippet の指定**
   - **箇所**: `build.yaml` 23行目 (`snippet: studio-rpc-usb-uart`)
   - **原因**: `studio-rpc-usb-uart` は ZMK `main` ブランチの新機能（ZMK Studio）であり、`v0.3-branch` には存在しないため、ビルド開始時にスニペット未存在エラーとなります。

2. **【原因2】`roBa.keymap` における未対応ヘッダーのインクルード**
   - **箇所**: `config/roBa.keymap` 1行目 (`#include <input/processors.dtsi>`) および 15行目 (`&mkp_input_listener`)
   - **原因**: Input Processors 機能は `main` ブランチ専用のため、`v0.3-branch` ではファイル未存在エラーとなります。

3. **【原因3】`roBa_R.overlay` における未対応 Input Processor ノードの定義**
   - **箇所**: `config/boards/shields/Test/roBa_R.overlay` 20-30行目 (`zip_temp_layer` / `trackball_listener`)
   - **原因**: `v0.3-branch` では `zmk,input-processor-temp-layer` の Binding が未定義です。`v0.3-branch` では `kumamuk-git/zmk-pmw3610-driver` ドライバーの標準機能（`mouse-layers = <3>;`）を使用する必要があります。

4. **【原因4】`roBa.keymap` における無効なキーコード指定**
   - **箇所**: `config/roBa.keymap` 211行目 (`&kp LA(LEFT)`)
   - **原因**: ZMK の標準キー定義（`keys.h`）に `LEFT` は存在せず、正しくは **`LEFT_ARROW`** です。C言語コンパイル時に `'LEFT' undeclared` エラーが発生します。

---

## 修正対応計画 (Proposed Changes)

### 1. `build.yaml` の修正
#### [MODIFY] [build.yaml](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/build.yaml)
- `snippet: studio-rpc-usb-uart` を削除。

### 2. `config/roBa.keymap` の修正
#### [MODIFY] [roBa.keymap](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/config/roBa.keymap)
- 1行目の `#include <input/processors.dtsi>` をコメントアウト。
- 15行目の `&mkp_input_listener` をコメントアウト。
- 211行目の `&kp LA(LEFT)` を **`&kp LA(LEFT_ARROW)`** に修正。

### 3. `config/boards/shields/Test/roBa_R.overlay` の修正
#### [MODIFY] [roBa_R.overlay](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/config/boards/shields/Test/roBa_R.overlay)
- `zip_temp_layer` および `trackball_listener` ノードをコメントアウト。
- `trackball` ノード内の **`mouse-layers = <3>;`** のコメントアウトを解除して有効化。

### 4. `config/boards/shields/Test/roBa_R.conf` の修正
#### [MODIFY] [roBa_R.conf](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/config/boards/shields/Test/roBa_R.conf)
- `v0.3-branch` 非対応の `CONFIG_ZMK_STUDIO=y` および `CONFIG_ZMK_STUDIO_LOCKING=n` をコメントアウト。

---

## 検証計画 (Verification Plan)
1. **コード修正と差分確認**: 上記4ファイルの修正を実施。
2. **Git Commit & Push**: 修正コミットを作成し、`origin/for-zonkey` へ Push。
3. **GitHub Actions 実行確認**: GitHub Actions の全ジョブ（`roBa_R`, `roBa_L`, `settings_reset`）が成功することを確認。

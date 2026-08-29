# GitHub Actions ビルドエラー (KeyError: 'qualifiers') 調査および解決計画

## 概要
GitHub Actions 上で ZMK ファームウェアのビルドを実行した際に発生する `KeyError: 'qualifiers'` エラーについて、解決策1（`.github/workflows/build.yml` の参照ブランチを `v0.3-branch` に固定）を採用して改修を実施します。

---

## エラーの根本原因（詳細解説）

### 1. 発生しているエラー
```text
  File "/tmp/zmk-config/zephyr/scripts/west_commands/boards.py", line 87, in do_run
    log.inf(args.format.format(name=board.name, arch=board.arch,
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'qualifiers'
```

### 2. 原因のメカニズム
1. **GitHub Actions ワークフロー側のバージョン (`.github/workflows/build.yml`)**:
   - `uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` が指定されていました。
   - ZMK の `main` ブランチでは、最新の Zephyr 3.7+ / 4.x (Hardware Model v2) への移行に伴い、`west boards` コマンドでボード情報をフォーマット出力する際に `{qualifiers}`（ボードの修飾子情報）を参照するようワークフロー内のスクリプトが更新されました。

2. **リポジトリ設定側のバージョン (`config/west.yml`)**:
   - 本リポジトリの `config/west.yml` では、ZMK のバージョンとして `revision: v0.3-branch` が指定されています。
   - `v0.3-branch` が依存している Zephyr 3.5 系の `boards.py` スクリプトは、引数のフォーマット文字列に `{qualifiers}` キーが存在することを想定しておらず、辞書展開時に `KeyError: 'qualifiers'` が発生して例外終了します。

3. **バージョンの不一致 (Version Mismatch)**:
   - **ワークフロー（親側）**: 最新の `main` ブランチの処理（新しい `west boards` コマンド構文を前提）
   - **ソースコード・依存関係（子側）**: `v0.3-branch`（古い `boards.py` を含む Zephyr）
   - このバージョンの食い違いによって、GitHub Actions がクラッシュしていました。

---

## 採用された解決策（解決策 1）
- [.github/workflows/build.yml](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/.github/workflows/build.yml) の参照先を `@v0.3-branch` に更新し、マニフェスト（`config/west.yml`）で指定された ZMK バージョンと完全に整合させます。

---

## 実施内容

### 変更ファイル: [.github/workflows/build.yml](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/.github/workflows/build.yml)

```diff
  on: [push, pull_request, workflow_dispatch]

  jobs:
    build:
-     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
+     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3-branch
```

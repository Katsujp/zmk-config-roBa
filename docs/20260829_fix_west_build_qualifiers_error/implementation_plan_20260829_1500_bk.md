# GitHub Actions ビルドエラー (KeyError: 'qualifiers') 調査および解決計画

## 概要
GitHub Actions 上で ZMK ファームウェアのビルドを実行した際に発生する `KeyError: 'qualifiers'` エラーについて、発生メカニズムをステップバイステップで解明し、確実な解決策を提示・実施します。

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
   - `uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` が指定されています。
   - ZMK の `main` ブランチでは、最新の Zephyr 3.7+ / 4.x (Hardware Model v2) への移行に伴い、`west boards` コマンドでボード情報をフォーマット出力する際に `{qualifiers}`（ボードの修飾子情報）を参照するようワークフロー内のスクリプトが更新されました。

2. **リポジトリ設定側のバージョン (`config/west.yml`)**:
   - 本リポジトリの `config/west.yml` では、ZMK のバージョンとして `revision: v0.3-branch` が指定されています。
   - `v0.3-branch` が依存している Zephyr 3.5 系の `boards.py` スクリプトは、引数のフォーマット文字列に `{qualifiers}` キーが存在することを想定しておらず、辞書展開時に `KeyError: 'qualifiers'` が発生して例外終了します。

3. **バージョンの不一致 (Version Mismatch)**:
   - **ワークフロー（親側）**: 最新の `main` ブランチの処理（新しい `west boards` コマンド構文を前提）
   - **ソースコード・依存関係（子側）**: `v0.3-branch`（古い `boards.py` を含む Zephyr）
   - このバージョンの食い違いによって、GitHub Actions がクラッシュしています。

---

## 解決策の比較

| 解決策 | 修正内容 | メリット | デメリット / 注意点 |
| :--- | :--- | :--- | :--- |
| **案1（推奨・即効性あり）**<br>ワークフローを `v0.3-branch` に固定 | `.github/workflows/build.yml` を `@v0.3-branch` または `@v0.3` に変更 | 既存のキーマップ・ドライバ・設定を変更することなく即座にビルドが成功する | 将来的な最新機能を利用するには別途移行が必要 |
| **案2（将来的な最新移行）**<br>ZMKリビジョンを最新 `main` に更新 | `config/west.yml` の `revision` を `main` に変更 | ZMKの最新機能が利用可能になる | Zephyr 4.x 移行に伴うボード/シールド記述やトラックボール用外部ドライバ（PMW3610）の互換性修正が必要になる可能性が高い |

---

## 修正手順（推奨案1の実施）

### 変更ファイル: [.github/workflows/build.yml](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/.github/workflows/build.yml)

```diff
  on: [push, pull_request, workflow_dispatch]

  jobs:
    build:
-     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
+     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3-branch
```

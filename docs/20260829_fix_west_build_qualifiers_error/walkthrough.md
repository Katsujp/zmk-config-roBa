# ビルドエラー調査および対応ウォークスルー (Walkthrough)

## 実施した調査内容
1. **GitHub Actions ワークフロー設定の確認**:
   - `.github/workflows/build.yml` にて `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` を参照していることを確認。
2. **West マニフェスト設定の確認**:
   - `config/west.yml` にて ZMK のブランチとして `v0.3-branch` が指定されていることを確認。
3. **Zephyr 側 `boards.py` との不整合確認**:
   - `KeyError: 'qualifiers'` は、ZMK `main` ブランチの最新ワークフローが Zephyr 3.7+ / 4.x の新しい `boards.py` 向けオプション（`{qualifiers}`）を指定して `west boards` を実行する一方、チェックアウトされる `v0.3-branch`（Zephyr 3.5系）の `boards.py` にそのキーが存在しないために発生することを特定。

## 実施した改修内容
- [.github/workflows/build.yml](file:///d:/User%20Files/Masashi/HOME/repos/zmk-config-roBa/.github/workflows/build.yml) の参照先を `@main` から `@v0.3-branch` に修正しました。

```diff
  on: [push, pull_request, workflow_dispatch]

  jobs:
    build:
-     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
+     uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3-branch
```

## 効果
- リポジトリの `config/west.yml` で指定されている `v0.3-branch` の ZMK / Zephyr 3.5 環境と、呼び出すワークフローのバージョンが一致します。
- `west boards` の実行時に不要な `{qualifiers}` オプションが渡されなくなり、`KeyError: 'qualifiers'` が解消され正常にファームウェアがビルドされます。

---
name: testflight
description: fastlane を使って TestFlight にビルドをアップロードし、テスターに配布します
disable-model-invocation: true
---

# TestFlight 配布スキル

fastlane を使用して iOS アプリのビルドを TestFlight にアップロードし、テスターグループに配布します。

## 前提条件の確認

以下の前提条件を確認してください:

1. **fastlane がインストールされていること**
   ```bash
   which fastlane
   ```

2. **Fastfile が存在すること**
   - プロジェクトルートの `fastlane/Fastfile` を確認

3. **Apple Developer アカウントの認証情報が設定されていること**
   - 環境変数 `FASTLANE_USER` または `APP_STORE_CONNECT_API_KEY_KEY_ID` 等が設定されていること

4. **有効な証明書とプロビジョニングプロファイルがあること**

## 実行手順

### ステップ 1: 環境の確認

```bash
# fastlane のバージョン確認
fastlane --version

# プロジェクトの Fastfile 確認
cat fastlane/Fastfile
```

### ステップ 2: ビルド番号のインクリメント

```bash
# 現在のビルド番号を確認
agvtool what-version

# ビルド番号をインクリメント
fastlane run increment_build_number
```

### ステップ 3: TestFlight へのアップロード

```bash
# TestFlight へアップロード
fastlane pilot upload
```

もし Fastfile に専用の lane がある場合:

```bash
# カスタム lane を使用
fastlane beta
```

### ステップ 4: テスターグループへの配布

```bash
# 特定のグループに配布
fastlane pilot distribute --groups "Internal Testers"
```

### ステップ 5: アップロード結果の確認

```bash
# 最新ビルドの確認
fastlane pilot builds
```

## トラブルシューティング

- **認証エラー**: `FASTLANE_USER` や App Store Connect API Key の設定を確認
- **証明書エラー**: `fastlane match` でプロビジョニングプロファイルを同期
- **ビルドエラー**: Xcode プロジェクトの設定（Bundle ID、チーム ID）を確認

## 引数

$ARGUMENTS が指定された場合、以下のオプションとして解釈します:

- `--group <グループ名>`: 配布先のテスターグループを指定
- `--skip-increment`: ビルド番号のインクリメントをスキップ
- `--lane <lane名>`: 使用する fastlane の lane を指定（デフォルト: `pilot upload`）

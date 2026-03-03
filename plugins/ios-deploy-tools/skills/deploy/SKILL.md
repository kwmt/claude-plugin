---
name: deploy
description: fastlane を使った iOS アプリの汎用デプロイ（testflight / appstore を引数で指定）
disable-model-invocation: true
---

# iOS 汎用デプロイスキル

fastlane を使用した iOS アプリのデプロイを実行します。引数で `testflight` または `appstore` を指定して、配布先を選択できます。

## 前提条件の確認

以下の前提条件をすべて確認してください:

### 1. fastlane のセットアップ確認

```bash
# fastlane がインストールされているか
which fastlane
fastlane --version

# Fastfile が存在するか
test -f fastlane/Fastfile && echo "Fastfile found" || echo "Fastfile not found"

# Appfile が存在するか
test -f fastlane/Appfile && echo "Appfile found" || echo "Appfile not found"
```

### 2. 環境変数の確認

```bash
# Apple ID / App Store Connect API Key の設定確認
echo "FASTLANE_USER: ${FASTLANE_USER:-(not set)}"
echo "APP_STORE_CONNECT_API_KEY_KEY_ID: ${APP_STORE_CONNECT_API_KEY_KEY_ID:-(not set)}"
echo "APP_STORE_CONNECT_API_KEY_ISSUER_ID: ${APP_STORE_CONNECT_API_KEY_ISSUER_ID:-(not set)}"
echo "APP_STORE_CONNECT_API_KEY_KEY: ${APP_STORE_CONNECT_API_KEY_KEY:+(set)}"
echo "MATCH_PASSWORD: ${MATCH_PASSWORD:-(not set)}"
```

### 3. 証明書・プロビジョニングプロファイルの確認

```bash
# match を使用している場合
fastlane match development --readonly
fastlane match appstore --readonly
```

### 4. Xcode プロジェクト設定の確認

```bash
# スキーム一覧
xcodebuild -list

# ビルド設定の確認
xcodebuild -showBuildSettings -scheme <SCHEME_NAME> | grep -E "(PRODUCT_BUNDLE_IDENTIFIER|DEVELOPMENT_TEAM|CODE_SIGN)"
```

## デプロイの実行

### 引数の解釈

`$ARGUMENTS` の最初の引数でデプロイ先を決定します:

- `testflight` → TestFlight にアップロード
- `appstore` → App Store Connect に提出
- 引数なし → ユーザーにデプロイ先を確認

### TestFlight デプロイ

```bash
# ビルド番号をインクリメント
fastlane run increment_build_number

# TestFlight にアップロード
fastlane pilot upload
```

### App Store デプロイ

```bash
# メタデータの確認
ls fastlane/metadata/

# バリデーション
fastlane deliver --skip_submission

# 提出
fastlane deliver --submit_for_review
```

### カスタム lane の使用

Fastfile にカスタム lane が定義されている場合はそちらを優先:

```bash
# Fastfile の内容を確認して適切な lane を選択
cat fastlane/Fastfile

# 例: beta lane がある場合
fastlane beta

# 例: release lane がある場合
fastlane release
```

## 実行後の確認

```bash
# TestFlight の場合: 最新ビルドを確認
fastlane pilot builds

# App Store の場合: ビルド番号を確認
fastlane run app_store_build_number
```

## トラブルシューティング

- **fastlane が見つからない**: `gem install fastlane` または `brew install fastlane` でインストール
- **Fastfile が存在しない**: `fastlane init` でセットアップを実行
- **認証エラー**: App Store Connect API Key の設定を確認。`fastlane spaceauth` でセッション更新
- **証明書エラー**: `fastlane match` でプロビジョニングプロファイルを同期
- **ビルドエラー**: `xcodebuild clean` を実行してからリトライ
- **コード署名エラー**: Xcode の Signing & Capabilities 設定を確認

## 引数

$ARGUMENTS が指定された場合、以下のように解釈します:

- 第1引数: デプロイ先（`testflight` または `appstore`）
- `--scheme <スキーム名>`: ビルドに使用する Xcode スキーム
- `--lane <lane名>`: 使用する fastlane の lane を直接指定
- `--skip-increment`: ビルド番号のインクリメントをスキップ
- `--dry-run`: 実際のデプロイを行わず、バリデーションのみ実行

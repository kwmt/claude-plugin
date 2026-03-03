---
name: appstore
description: fastlane を使って App Store Connect にビルドを提出します
disable-model-invocation: true
---

# App Store 提出スキル

fastlane を使用して iOS アプリのビルドを App Store Connect に提出し、審査に送信します。

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

4. **有効な証明書（Distribution）とプロビジョニングプロファイルがあること**

5. **App Store Connect 上でアプリが作成済みであること**

## 実行手順

### ステップ 1: 環境の確認

```bash
# fastlane のバージョン確認
fastlane --version

# プロジェクトの Fastfile 確認
cat fastlane/Fastfile
```

### ステップ 2: メタデータとスクリーンショットの確認

```bash
# メタデータの確認
ls fastlane/metadata/

# スクリーンショットの確認
ls fastlane/screenshots/
```

メタデータが存在しない場合:

```bash
# App Store Connect からメタデータをダウンロード
fastlane deliver download_metadata
fastlane deliver download_screenshots
```

### ステップ 3: 提出前のバリデーション

```bash
# ビルドのバリデーション（アップロードせずに検証のみ）
fastlane deliver --skip_submission --skip_screenshots --precheck_include_in_app_purchases false
```

### ステップ 4: App Store への提出

```bash
# App Store にビルドを提出
fastlane deliver --submit_for_review
```

もし Fastfile に専用の lane がある場合:

```bash
# カスタム lane を使用
fastlane release
```

### ステップ 5: 提出結果の確認

App Store Connect で審査ステータスを確認してください:
- https://appstoreconnect.apple.com

```bash
# fastlane spaceship で審査ステータスを確認（API Key 認証が必要）
fastlane run app_store_build_number
```

## トラブルシューティング

- **メタデータエラー**: `fastlane/metadata/` ディレクトリの内容を確認し、必須フィールドが埋まっているか確認
- **スクリーンショットエラー**: 必要なデバイスサイズのスクリーンショットが揃っているか確認
- **審査リジェクト**: App Store Review ガイドラインを確認し、指摘事項を修正
- **バイナリ拒否**: アーキテクチャ、最小 OS バージョン、プライバシーマニフェストを確認

## 引数

$ARGUMENTS が指定された場合、以下のオプションとして解釈します:

- `--skip-metadata`: メタデータの更新をスキップ
- `--skip-screenshots`: スクリーンショットの更新をスキップ
- `--validate-only`: バリデーションのみ実行（提出しない）
- `--lane <lane名>`: 使用する fastlane の lane を指定（デフォルト: `deliver`）

# iOS Deploy Tools - Claude Code Plugin

fastlane を活用した iOS アプリの TestFlight 配布・App Store 提出を自動化する Claude Code プラグインです。

## インストール

### マーケットプレイスとして追加

```
/plugin marketplace add kwmt/claude-plugine
```

### プラグインのインストール

```
/plugin install ios-deploy-tools@kwmt-ios-tools
```

## スキル一覧

| スキル | コマンド | 説明 |
|--------|---------|------|
| TestFlight 配布 | `/ios-deploy-tools:testflight` | TestFlight にビルドをアップロードし、テスターに配布 |
| App Store 提出 | `/ios-deploy-tools:appstore` | App Store Connect にビルドを提出 |
| 汎用デプロイ | `/ios-deploy-tools:deploy` | 引数で配布先を指定して実行 |

## 使い方

### TestFlight に配布

```
/ios-deploy-tools:testflight
```

特定のテスターグループに配布:

```
/ios-deploy-tools:testflight --group "Internal Testers"
```

### App Store に提出

```
/ios-deploy-tools:appstore
```

バリデーションのみ実行:

```
/ios-deploy-tools:appstore --validate-only
```

### 汎用デプロイ

```
/ios-deploy-tools:deploy testflight
/ios-deploy-tools:deploy appstore
/ios-deploy-tools:deploy testflight --dry-run
```

## 前提条件

- **fastlane**: `gem install fastlane` または `brew install fastlane`
- **Xcode**: App Store からインストール
- **Apple Developer アカウント**: App Store Connect API Key または Apple ID 認証の設定
- **証明書・プロビジョニングプロファイル**: `fastlane match` 等で管理

## 環境変数

以下の環境変数を設定してください（App Store Connect API Key 認証の場合）:

| 変数名 | 説明 |
|--------|------|
| `APP_STORE_CONNECT_API_KEY_KEY_ID` | API Key ID |
| `APP_STORE_CONNECT_API_KEY_ISSUER_ID` | Issuer ID |
| `APP_STORE_CONNECT_API_KEY_KEY` | API Key の内容（.p8 ファイル） |

または Apple ID 認証の場合:

| 変数名 | 説明 |
|--------|------|
| `FASTLANE_USER` | Apple ID |
| `FASTLANE_PASSWORD` | Apple ID パスワード |

## ローカル開発

### プラグインのローカルテスト

```bash
claude --plugin-dir ./plugins/ios-deploy-tools
```

### バリデーション

```bash
claude plugin validate .
```

## ライセンス

MIT License - 詳細は [LICENSE](LICENSE) を参照してください。

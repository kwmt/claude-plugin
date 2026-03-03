# iOS Deploy Tools - Claude Code Plugin

fastlane を活用した iOS アプリのセットアップ・ビルド・デプロイを自動化する Claude Code プラグインです。

## インストール

### マーケットプレイスとして追加

```
/plugin marketplace add kwmt/claude-plugin
```

### プラグインのインストール

```
/plugin install ios-deploy-tools@kwmt-tools
```

## スキル一覧

### 初期セットアップ（プロジェクト初回のみ）

| スキル | コマンド | 説明 |
|--------|---------|------|
| fastlane 初期化 | `/ios-deploy-tools:ios-init-fastlane` | Gemfile, Appfile, Fastfile の雛形を生成 |
| 証明書管理 | `/ios-deploy-tools:ios-init-match` | fastlane match で証明書・プロファイルをセットアップ |
| CI 構築 | `/ios-deploy-tools:ios-init-ci` | GitHub Actions ワークフローを生成 |

### 開発・検証（随時）

| スキル | コマンド | 説明 |
|--------|---------|------|
| ローカルビルド | `/ios-deploy-tools:ios-build-local` | Development / Release ビルドの検証 |

### リリース（リリースごと）

| スキル | コマンド | 説明 |
|--------|---------|------|
| TestFlight 配信 | `/ios-deploy-tools:ios-deploy-testflight` | タグ・Release 作成 → TestFlight 配布 |
| Firebase 配信 | `/ios-deploy-tools:ios-deploy-firebase` | Firebase App Distribution で配信 |
| App Store 提出 | `/ios-deploy-tools:ios-deploy-appstore` | App Store Review に提出 |

## 依存関係

スキルの推奨実行順序:

```
ios-init-fastlane → ios-init-match → ios-init-ci → ios-build-local / ios-deploy-*
```

1. **ios-init-fastlane**: まず fastlane をセットアップ
2. **ios-init-match**: 証明書・プロファイルを管理（fastlane が必要）
3. **ios-init-ci**: CI/CD ワークフローを構築（fastlane + match が必要）
4. **ios-build-local / ios-deploy-***: ビルド・デプロイを実行

## 使い方

### 初回セットアップ

```
/ios-deploy-tools:ios-init-fastlane
/ios-deploy-tools:ios-init-match
/ios-deploy-tools:ios-init-ci
```

### ローカルビルド検証

```
/ios-deploy-tools:ios-build-local
```

### TestFlight に配信

```
/ios-deploy-tools:ios-deploy-testflight
```

### Firebase App Distribution に配信

```
/ios-deploy-tools:ios-deploy-firebase
```

### App Store に提出

```
/ios-deploy-tools:ios-deploy-appstore
```

## 前提条件

- **fastlane**: `gem install fastlane` または `brew install fastlane`
- **Xcode**: App Store からインストール
- **Apple Developer アカウント**: App Store Connect API Key の設定
- **rbenv**: Ruby バージョン管理
- **GitHub CLI (`gh`)**: GitHub Actions の実行・Release 作成に使用

## 環境変数

### App Store Connect API Key（必須）

| 変数名 | 説明 |
|--------|------|
| `APP_STORE_CONNECT_API_KEY_KEY_ID` | API Key ID |
| `APP_STORE_CONNECT_API_KEY_ISSUER_ID` | Issuer ID |
| `APP_STORE_CONNECT_API_KEY_KEY` | API Key の内容（Base64 エンコード済み .p8） |

### match（証明書管理）

| 変数名 | 説明 |
|--------|------|
| `MATCH_PASSWORD` | match 暗号化パスフレーズ |
| `MATCH_GIT_URL` | 証明書リポジトリの URL |
| `MATCH_GIT_BASIC_AUTHORIZATION` | リポジトリ認証（Base64） |

### Firebase（Firebase 配信時のみ）

| 変数名 | 説明 |
|--------|------|
| `FIREBASE_APP_ID` | Firebase App ID |
| `FIREBASE_TOKEN` | Firebase CLI 認証トークン |

## ローカル開発

### プラグインのローカルテスト

```bash
claude --plugin-dir ./plugins/ios-deploy-tools
```

### バリデーション

```bash
claude plugin validate ./plugins/ios-deploy-tools
claude plugin validate .
```

## ライセンス

MIT License - 詳細は [LICENSE](LICENSE) を参照してください。

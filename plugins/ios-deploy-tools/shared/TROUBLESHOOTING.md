# 共通トラブルシューティングガイド

## 共通エラー対応ルール（全スキル共通の行動指針）

- コマンドが失敗したら、エラー出力を分析し **原因** と **複数の解決方法** をユーザーに提示する
- 解決方法は具体的なコマンド付きで提示し、ユーザーに選択を求める
- ユーザーが選んだ方法を実行し、それも失敗したら次の対応策を提示する（繰り返し）
- 自分だけでは判断できない場合は正直にその旨を伝える

## よくあるエラーパターン集

### 証明書・プロファイル

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "doesn't include signing certificate" | プロファイルと証明書の不一致 | A) `match appstore --force` で再取得 B) `match nuke distribution` → `match appstore` で再作成 |
| "No matching provisioning profile found" | プロファイル未生成または期限切れ | A) `match development/appstore` を再実行 B) Apple Developer Portal で手動確認 |
| Code signing エラー全般 | 証明書・プロファイル・チーム設定の問題 | A) match を再実行 B) Xcode で手動設定確認を案内 |

### 認証

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| 2FA / MFA 認証エラー | 非対話的環境で2FAが必要 | A) App Store Connect API Key を設定（推奨） B) ターミナルで手動実行 |
| "Invalid API Key" / API Key 関連エラー | 環境変数未設定・不正 | 3つの環境変数 (`APP_STORE_CONNECT_API_KEY_KEY_ID`, `APP_STORE_CONNECT_API_KEY_ISSUER_ID`, `APP_STORE_CONNECT_API_KEY_KEY`) の設定状況を確認・案内 |

### Ruby 環境

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "rbenv: version not installed" | 指定 Ruby バージョン未インストール | A) `rbenv install <version>` B) `.ruby-version` を既存バージョンに変更 |
| "bundle: command not found" / Gem 関連エラー | bundler 未インストール | `gem install bundler && bundle install` |

### Keychain

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "User interaction is not allowed" | CI 環境でキーチェーンアクセス不可 | A) `setup_ci` を Fastfile に追加 B) キーチェーンのロック解除コマンド |
| "Matchfile: keychain_password" 警告 | Matchfile で値が未設定 | `MATCH_KEYCHAIN_PASSWORD` 環境変数を設定、または Matchfile から該当行を削除 |

### match

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| 証明書リポジトリアクセス拒否 | SSH/HTTPS の認証情報が不正 | A) SSH キーの確認・再設定 B) HTTPS + PAT に切り替え |
| パスフレーズエラー | `MATCH_PASSWORD` が不一致 | A) 正しいパスフレーズで再設定 B) `match nuke` で証明書を再作成 |

### ビルド

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "xcodebuild: error: ... scheme not found" | スキーム名の不一致 | スキーム一覧を表示して正しいスキーム名を選択 |
| xcodebuild タイムアウト | ビルド時間超過 | A) タイムアウト値を延長 B) クリーンビルド (`xcodebuild clean`) |

### Firebase

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "FIREBASE_APP_ID is not set" | 環境変数未設定 | Firebase Console から App ID を確認・設定する手順を案内 |
| Firebase token 期限切れ | トークンの有効期限切れ | `firebase login:ci` で再取得 |

### TestFlight

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "Build number already exists" | ビルド番号の重複 | A) `build_number` オプションで明示指定 B) `latest_testflight_build_number + 1` を確認 |

### App Store

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| メタデータエラー | App Store Connect の情報不足 | エラー内容に応じて App Store Connect での修正箇所を案内 |
| "No build found" | TestFlight にビルドが存在しない | TestFlight ビルドの存在確認と再アップロード手順を案内 |

### CI (GitHub Actions)

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| ワークフロー構文エラー | YAML の書式不正 | YAML バリデーションツールで構文チェック |
| Secrets 未設定による実行失敗 | GitHub Secrets が未登録 | 必要な Secrets の設定チェックリストを再表示 |
| ワークフロー実行失敗 | 各種原因 | `gh run view <run_id> --log-failed` でログ確認手順を案内 |

---
name: ios-init-match
description: fastlane match で証明書・プロビジョニングプロファイルを初期セットアップする
disable-model-invocation: true
---

fastlane match で証明書・プロビジョニングプロファイルを初期セットアップする。証明書リポジトリの作成と GitHub Secrets の設定案内を行う。

## 手順

1. `fastlane/Appfile` が存在するか確認する。存在しない場合は `/ios-init-fastlane` を先に実行するよう案内して中止する。
2. ユーザーに証明書リポジトリの情報を確認する:
   - 新規作成するか、既存リポジトリを使うか
   - リポジトリ URL（例: `git@github.com:<org>/<project>-certificates.git`）
3. 新規作成の場合:
   ```bash
   gh repo create <project>-certificates --private
   ```
4. `fastlane/Matchfile` を作成する:
   ```ruby
   git_url(ENV["MATCH_GIT_URL"] || "<certificates_repo_url>")
   storage_mode("git")
   type("appstore")
   app_identifier(["<app_identifier>"])
   team_id("<team_id>")
   ```
5. App Store Connect API Key の設定方法を案内する:
   - Apple Developer → Users and Access → Integrations → App Store Connect API
   - API キーを作成し、Key ID, Issuer ID, .p8 ファイルを取得
   - ローカル環境変数の設定:
     ```bash
     export APP_STORE_CONNECT_API_KEY_KEY_ID="YOUR_KEY_ID"
     export APP_STORE_CONNECT_API_KEY_ISSUER_ID="YOUR_ISSUER_ID"
     export APP_STORE_CONNECT_API_KEY_KEY="$(base64 < AuthKey_XXXXXX.p8)"
     ```
6. match を実行して証明書を生成する:
   ```bash
   bundle exec fastlane match development
   bundle exec fastlane match appstore
   ```
   - パスフレーズを設定した場合は記録しておくよう案内する（`MATCH_PASSWORD` に使用）。
7. GitHub Secrets の設定チェックリストを表示する:

   | Secret 名 | 説明 | 設定方法 |
   |-----------|------|---------|
   | `MATCH_PASSWORD` | match 暗号化パスフレーズ | match init 時に設定したもの |
   | `MATCH_GIT_URL` | 証明書リポジトリの HTTPS URL | `https://github.com/<org>/<project>-certificates.git` |
   | `MATCH_GIT_BASIC_AUTHORIZATION` | リポジトリ認証 | `echo -n "username:PAT" \| base64` |
   | `APP_STORE_CONNECT_API_KEY_KEY_ID` | API Key ID | App Store Connect で確認 |
   | `APP_STORE_CONNECT_API_KEY_ISSUER_ID` | Issuer ID | App Store Connect で確認 |
   | `APP_STORE_CONNECT_API_KEY_KEY` | API Key 本体 | `base64 < AuthKey_XXXXXX.p8` |

8. 結果を報告する:
   - 作成した証明書・プロファイルの一覧
   - 設定が必要な GitHub Secrets のチェックリスト
   - 次のステップ: `/ios-init-ci` で CI パイプラインを構築

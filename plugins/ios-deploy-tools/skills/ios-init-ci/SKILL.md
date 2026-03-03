---
name: ios-init-ci
description: iOS の CI/CD パイプラインとなる GitHub Actions ワークフローを構築する
disable-model-invocation: true
---

iOS の CI/CD パイプラインとなる GitHub Actions ワークフローを構築する。CI, TestFlight, Firebase, App Store 用のワークフローを生成する。

## 手順

1. 前提条件を確認する:
   - `fastlane/Fastfile` が存在するか
   - `fastlane/Matchfile` が存在するか
   - 存在しない場合は `/ios-init-fastlane` と `/ios-init-match` を先に実行するよう案内して中止する。
   - `.github/workflows/` ディレクトリが存在するか（なければ `mkdir -p .github/workflows` で作成）
2. ユーザーに以下を確認する:
   - iOS プロジェクトのパス（リポジトリルートからの相対パス、例: `ios/`, `client/iosApp/`）
   - iOS ソースの変更検出パス（CI paths フィルタ用、例: `ios/**`）
   - Ruby バージョン（`.ruby-version` から自動検出、例: `3.3`）
   - Java が必要か（KMP プロジェクトの場合、Java バージョンも確認）
3. `.github/workflows/ci-ios.yml` を作成する（PR 時の iOS ビルド検証）:
   - トリガー: `pull_request` (paths フィルタ) + `workflow_dispatch`
   - ランナー: `macos-latest`
   - ステップ: checkout → Ruby setup → bundle install → xcodebuild（シミュレータビルド）
   - KMP プロジェクトの場合は Java setup を追加する
4. `.github/workflows/deploy-ios-testflight.yml` を作成する:
   - トリガー: `workflow_dispatch`（build_number オプション入力）
   - ランナー: `macos-latest`
   - Secrets: MATCH_PASSWORD, MATCH_GIT_URL, MATCH_GIT_BASIC_AUTHORIZATION, APP_STORE_CONNECT_API_KEY_*
   - ステップ: checkout → Java setup（KMP の場合）→ Ruby setup → bundle install → `bundle exec fastlane beta`
   - 環境変数:
     ```yaml
     env:
       MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
       MATCH_GIT_URL: ${{ secrets.MATCH_GIT_URL }}
       MATCH_GIT_BASIC_AUTHORIZATION: ${{ secrets.MATCH_GIT_BASIC_AUTHORIZATION }}
       APP_STORE_CONNECT_API_KEY_KEY_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_KEY_ID }}
       APP_STORE_CONNECT_API_KEY_ISSUER_ID: ${{ secrets.APP_STORE_CONNECT_API_KEY_ISSUER_ID }}
       APP_STORE_CONNECT_API_KEY_KEY: ${{ secrets.APP_STORE_CONNECT_API_KEY_KEY }}
     ```
5. `.github/workflows/deploy-ios-firebase.yml` を作成する:
   - トリガー: `workflow_dispatch`（groups, release_notes オプション入力）
   - ランナー: `macos-latest`
   - Secrets: 上記 + FIREBASE_APP_ID, FIREBASE_TOKEN
   - ステップ: checkout → Java setup（KMP の場合）→ Ruby setup → bundle install → `bundle exec fastlane distribute_firebase`
   - 追加の環境変数:
     ```yaml
     env:
       FIREBASE_APP_ID: ${{ secrets.FIREBASE_APP_ID }}
       FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
     ```
6. `.github/workflows/deploy-ios-appstore.yml` を作成する:
   - トリガー: `workflow_dispatch`（submit_for_review, automatic_release ブール入力）
   - ランナー: `macos-latest`
   - Secrets: APP_STORE_CONNECT_API_KEY_*
   - ステップ: checkout → Ruby setup → bundle install → `bundle exec fastlane release`
   - ビルド不要（skip_binary_upload: true で TestFlight の既存ビルドを使用）
7. 結果を報告する:
   - 作成したワークフローファイル一覧
   - 設定が必要な GitHub Secrets 一覧:

     | Secret | 用途 | 使用ワークフロー |
     |--------|------|-----------------|
     | `MATCH_PASSWORD` | match 暗号化パスフレーズ | TestFlight, Firebase |
     | `MATCH_GIT_URL` | 証明書リポジトリ URL | TestFlight, Firebase |
     | `MATCH_GIT_BASIC_AUTHORIZATION` | 証明書リポジトリ認証 (Base64) | TestFlight, Firebase |
     | `APP_STORE_CONNECT_API_KEY_KEY_ID` | App Store Connect API Key ID | TestFlight, Firebase, AppStore |
     | `APP_STORE_CONNECT_API_KEY_ISSUER_ID` | App Store Connect Issuer ID | TestFlight, Firebase, AppStore |
     | `APP_STORE_CONNECT_API_KEY_KEY` | App Store Connect API Key (Base64) | TestFlight, Firebase, AppStore |
     | `FIREBASE_APP_ID` | Firebase App ID | Firebase |
     | `FIREBASE_TOKEN` | Firebase CLI 認証トークン | Firebase |

   - 各ワークフローの手動実行方法:
     ```bash
     gh workflow run ci-ios.yml
     gh workflow run deploy-ios-testflight.yml
     gh workflow run deploy-ios-firebase.yml -f groups="testers" -f release_notes="..."
     gh workflow run deploy-ios-appstore.yml -f submit_for_review=false -f automatic_release=false
     ```

## エラー対応

コマンド実行でエラーが発生した場合は、`shared/TROUBLESHOOTING.md` を参照しつつ以下の手順で対応する:

1. エラー出力を分析し、**原因** を特定する
2. **複数の解決方法** を具体的なコマンド付きで提示する
3. ユーザーにどの方法で進めるか **選択を求める**
4. 選択された方法を実行する
5. それでも失敗した場合は、別の解決方法を提示して再度選択を求める

### このスキル固有のよくあるエラー

- **ワークフロー構文エラー** → A) `actionlint` や YAML バリデーターで構文チェック B) GitHub Actions のエラーログから該当行を特定して修正
- **Secrets 未設定による実行失敗** → 必要な Secrets の設定チェックリストを再表示し、`gh secret list` で設定状況を確認
- **ワークフロー実行失敗** → `gh run view <run_id> --log-failed` でエラーログを確認し、原因に応じた解決策を提示

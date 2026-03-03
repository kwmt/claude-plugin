---
name: ios-deploy-appstore
description: TestFlight の最新ビルドを App Store Review に提出する
disable-model-invocation: true
---

TestFlight の最新ビルドを App Store Review に提出する。GitHub Actions またはローカルで fastlane を使って提出を行う。

## 手順

1. 現在のブランチが `main` であることを確認する。`main` でない場合はエラーメッセージを表示して中止する。
2. 最新の iOS タグを確認する:
   - `git tag -l "ios/v*" --sort=-version:refname | head -1` で最新タグを取得する。
   - 対応する GitHub Release の URL を表示して、これが提出対象で正しいか確認する。
3. ユーザーに提出オプションを確認する:
   - `submit_for_review` — 審査に自動提出するか（デフォルト: false）
   - `automatic_release` — 審査通過後に自動リリースするか（デフォルト: false）
4. デプロイ方法を選択する:
   - **GitHub Actions（推奨）**:
     ```bash
     gh workflow run deploy-ios-appstore.yml -f submit_for_review=<bool> -f automatic_release=<bool>
     ```
     - `gh run list --workflow=deploy-ios-appstore.yml --limit=1` でワークフロー起動を確認する。
   - **ローカル実行**: fastlane を直接実行する:
     ```bash
     cd <ios_project_path>
     eval "$(rbenv init -)" && rbenv shell $(cat .ruby-version)
     bundle install
     bundle exec fastlane release submit_for_review:<bool> automatic_release:<bool>
     ```
5. 結果を報告する:
   - 提出対象のビルドバージョン
   - 審査状況の確認方法（App Store Connect URL）
   - デプロイ方法と実行結果（成功 or エラー内容）

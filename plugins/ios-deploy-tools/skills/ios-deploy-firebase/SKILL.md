---
name: ios-deploy-firebase
description: iOS アプリを Firebase App Distribution で配信する
disable-model-invocation: true
---

iOS アプリを Firebase App Distribution で配信する。GitHub Actions またはローカルで fastlane を使ってビルド・配信を行う。

## 手順

1. 現在のブランチを確認する。配信元のブランチを確認（main 以外からの配信も許可）。
2. `git fetch origin && git pull origin <ブランチ名>` でローカルを最新化する。
3. ユーザーに以下を確認する:
   - テスターグループ名（デフォルト: `testers`）
   - リリースノート（空欄可）
4. デプロイ方法を選択する:
   - **GitHub Actions（推奨）**:
     ```bash
     gh workflow run deploy-ios-firebase.yml -f groups=<groups> -f release_notes="<notes>"
     ```
     - `gh run list --workflow=deploy-ios-firebase.yml --limit=1` でワークフロー起動を確認する。
   - **ローカル実行**: fastlane を直接実行する:
     ```bash
     cd <ios_project_path>
     eval "$(rbenv init -)" && rbenv shell $(cat .ruby-version)
     bundle install
     bundle exec fastlane distribute_firebase groups:"<groups>" release_notes:"<notes>"
     ```
     - `FIREBASE_APP_ID` 環境変数が設定されていることを確認する。
     - ビルドには時間がかかるため、タイムアウトを長めに設定する（10分）。
5. 結果を報告する:
   - 配信先のテスターグループ
   - デプロイ方法と実行結果（成功 or エラー内容）

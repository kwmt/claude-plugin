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
   - デプロイ方法と実行結果

## エラー対応

コマンド実行でエラーが発生した場合は、`shared/TROUBLESHOOTING.md` を参照しつつ以下の手順で対応する:

1. エラー出力を分析し、**原因** を特定する
2. **複数の解決方法** を具体的なコマンド付きで提示する
3. ユーザーにどの方法で進めるか **選択を求める**
4. 選択された方法を実行する
5. それでも失敗した場合は、別の解決方法を提示して再度選択を求める

### このスキル固有のよくあるエラー

- **`FIREBASE_APP_ID` 未設定** → Firebase Console (プロジェクト設定 → マイアプリ) から App ID を確認し、環境変数に設定する手順を案内
- **Firebase token 期限切れ** → `firebase login:ci` を実行してトークンを再取得し、環境変数 `FIREBASE_TOKEN` を更新
- **テスターグループが存在しない** → Firebase Console (App Distribution → テスターとグループ) でグループを作成する手順を案内

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
   - デプロイ方法と実行結果

## エラー対応

コマンド実行でエラーが発生した場合は、`shared/TROUBLESHOOTING.md` を参照しつつ以下の手順で対応する:

1. エラー出力を分析し、**原因** を特定する
2. **複数の解決方法** を具体的なコマンド付きで提示する
3. ユーザーにどの方法で進めるか **選択を求める**
4. 選択された方法を実行する
5. それでも失敗した場合は、別の解決方法を提示して再度選択を求める

### このスキル固有のよくあるエラー

- **メタデータエラー** → エラーメッセージを分析し、App Store Connect で修正が必要な箇所（スクリーンショット、説明文、プライバシーポリシー等）を具体的に案内
- **"No build found"** → A) TestFlight にビルドが存在するか確認 B) ビルドの処理完了を待つ C) `/ios-deploy-testflight` で再アップロード
- **審査提出失敗** → エラー内容に応じて App Store Connect での操作手順を案内（Export Compliance、コンテンツ権利等）

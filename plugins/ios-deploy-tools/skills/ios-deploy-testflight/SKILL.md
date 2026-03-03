---
name: ios-deploy-testflight
description: iOS TestFlight 用のタグ・GitHub Release を作成し、ビルド・配布する（汎用版）
disable-model-invocation: true
---

iOS TestFlight 用のタグと GitHub Release を作成し、GitHub Actions またはローカルで fastlane を使ってビルド・TestFlight 配布を行う。

## 手順

1. 現在のブランチを確認する。`main` でない場合は `git checkout main` で移動する。
2. `git fetch origin && git pull origin main` でローカルを最新化する。
3. 今日の日付（YYYY-MM-DD 形式）を取得する。
4. 既存のタグを確認して連番を自動決定する:
   - `git tag -l "ios/v<今日の日付>.*"` で今日作成済みのタグを一覧取得
   - 最大の連番 + 1 を次の番号とする（なければ 1）
5. タグ名: `ios/v<今日の日付>.<連番>`（例: `ios/v2026-03-01.1`）
6. `git tag <タグ名> && git push origin <タグ名>` でタグを作成・プッシュする。
7. GitHub Release を作成する:
   - `git tag -l "ios/v*" --sort=-version:refname | grep -v <今回のタグ名> | head -1` で前回の iOS タグを取得する。
   - 前回タグと今回タグの間の PR から changelog を生成する。
   - 以下のスクリプトで変更があった PR を抽出する:
     ```bash
     NOTES=""
     for merge in $(git log --merges --first-parent --format="%H" <前回タグ>..<今回タグ>); do
       pr_num=$(git log --format="%s" -1 "$merge" | grep -oE '#[0-9]+' | head -1)
       if [ -n "$pr_num" ]; then
         pr_title=$(gh pr view "${pr_num#\#}" --json title --jq '.title')
         NOTES="${NOTES}* ${pr_title} (${pr_num})\n"
       fi
     done
     ```
   - 該当 PR がある場合: `## What's Changed` ヘッダー + PR 一覧 + Full Changelog リンクを `--notes` で渡す。
   - 該当 PR がない場合: `--notes "変更はありません。\n\n**Full Changelog**: <比較URL>"` を渡す。
   - `--latest` は付けない。
8. デプロイ方法を選択する:
   - **GitHub Actions（推奨）**: `gh workflow run deploy-ios-testflight.yml` を実行する。
     - `gh run list --workflow=deploy-ios-testflight.yml --limit=1` でワークフロー起動を確認する。
   - **ローカル実行**: fastlane を直接実行する:
     ```bash
     cd <ios_project_path>
     eval "$(rbenv init -)" && rbenv shell $(cat .ruby-version)
     bundle install
     bundle exec fastlane beta
     ```
     - ビルドには時間がかかるため、タイムアウトを長めに設定する（10分）。
9. 結果を報告する:
   - 作成したタグ名
   - Release URL
   - デプロイ方法と実行結果（成功 or エラー内容）

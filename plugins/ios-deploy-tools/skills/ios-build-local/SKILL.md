---
name: ios-build-local
description: ローカル環境で iOS アプリのビルド検証を行う
disable-model-invocation: true
---

ローカル環境で iOS アプリのビルド検証を行う。Development ビルドまたは Release ビルドを選択して実行する。

## 手順

1. iOS プロジェクトのルートディレクトリを特定する（`.xcodeproj` の場所）。`find . -name "*.xcodeproj" -maxdepth 3` で検索する。
2. Ruby 環境をセットアップする:
   ```bash
   eval "$(rbenv init -)" && rbenv shell $(cat .ruby-version)
   bundle install
   ```
3. ビルドの種類をユーザーに確認する:
   - **Development** — シミュレータ/実機向けビルド
   - **Release** — App Store 用 IPA 生成
4. Development ビルドの場合:
   ```bash
   bundle exec fastlane match_development
   xcodebuild -project <project>.xcodeproj -scheme <debug_scheme> -sdk iphonesimulator build
   ```
   - スキーム名は `ls *.xcodeproj/xcshareddata/xcschemes/` で確認する。
5. Release ビルドの場合:
   ```bash
   bundle exec fastlane build_release
   ```
   - ビルドには時間がかかるため、タイムアウトを長めに設定する（10分）。
   - 成功すると `./build/<app_name>.ipa` が生成される。
6. 結果を報告する:
   - ビルド成功/失敗
   - 出力物のパスとサイズ

## エラー対応

コマンド実行でエラーが発生した場合は、`shared/TROUBLESHOOTING.md` を参照しつつ以下の手順で対応する:

1. エラー出力を分析し、**原因** を特定する
2. **複数の解決方法** を具体的なコマンド付きで提示する
3. ユーザーにどの方法で進めるか **選択を求める**
4. 選択された方法を実行する
5. それでも失敗した場合は、別の解決方法を提示して再度選択を求める

### このスキル固有のよくあるエラー

- **スキーム名が見つからない** → `ls *.xcodeproj/xcshareddata/xcschemes/` でスキーム一覧を表示し、正しいスキーム名をユーザーに選択してもらう
- **プロビジョニングプロファイルエラー** → A) `match development/appstore` を再実行 B) `match appstore --force` で強制再取得
- **xcodebuild タイムアウト** → A) タイムアウト値を延長して再実行 B) `xcodebuild clean` 後にクリーンビルド

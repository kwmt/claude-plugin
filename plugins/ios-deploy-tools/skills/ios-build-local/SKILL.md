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
   - エラーがあった場合は原因と解決策

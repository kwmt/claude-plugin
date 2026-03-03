---
name: ios-init-fastlane
description: 新規 iOS プロジェクトに fastlane を初期セットアップする（Gemfile, Appfile, Fastfile 生成）
disable-model-invocation: true
---

新規 iOS プロジェクトに fastlane を初期セットアップする。Gemfile, Appfile, Fastfile の雛形を生成する。

## 手順

1. iOS プロジェクトルート（`.xcodeproj` が存在するディレクトリ）を特定する。`find . -name "*.xcodeproj" -maxdepth 3` で検索する。
2. `.ruby-version` を確認する。存在しない場合は `rbenv local 3.3.6` で作成する。
3. `Gemfile` が存在しない場合、以下の内容で作成する:
   ```ruby
   source "https://rubygems.org"
   gem "fastlane"
   gem "fastlane-plugin-firebase_app_distribution"
   ```
4. `bundle install` を実行する。
5. ユーザーに以下を確認する:
   - `app_identifier`（例: `com.example.app`）
   - `team_id`（Apple Developer Team ID）
   - Xcode プロジェクト名（自動検出した `.xcodeproj` 名を提示）
   - Release 用スキーム名（`ls *.xcodeproj/xcshareddata/xcschemes/` で検出・提示）
   - Release 用 Configuration 名（例: `Release`, `Release-Production`）
   - アプリ名（IPA 出力名）
6. `fastlane/Appfile` を作成する:
   ```ruby
   app_identifier("<app_identifier>")
   team_id("<team_id>")
   ```
7. `fastlane/Fastfile` を以下のテンプレートで作成する（ユーザー回答でプレースホルダを置換）:
   ```ruby
   default_platform(:ios)

   platform :ios do
     private_lane :fetch_api_key do
       if ENV["APP_STORE_CONNECT_API_KEY_KEY_ID"]
         app_store_connect_api_key(
           key_id: ENV["APP_STORE_CONNECT_API_KEY_KEY_ID"],
           issuer_id: ENV["APP_STORE_CONNECT_API_KEY_ISSUER_ID"],
           key_content: ENV["APP_STORE_CONNECT_API_KEY_KEY"],
           is_key_content_base64: true,
         )
       end
     end

     desc "Development 証明書・プロファイル取得"
     lane :match_development do
       fetch_api_key
       match(type: "development", readonly: is_ci)
     end

     desc "App Store 証明書・プロファイル取得"
     lane :match_appstore do
       fetch_api_key
       match(type: "appstore", readonly: is_ci)
     end

     desc "Release ビルド"
     lane :build_release do |options|
       setup_ci if is_ci
       fetch_api_key

       if options[:build_number]
         increment_build_number(build_number: options[:build_number])
       else
         latest = latest_testflight_build_number
         new_build_number = latest + 1
         UI.message("Build number: #{latest} → #{new_build_number}")
         increment_build_number(build_number: new_build_number)
       end

       match(type: "appstore", readonly: true)

       build_app(
         project: "<project_name>.xcodeproj",
         scheme: "<release_scheme>",
         configuration: "<release_configuration>",
         export_method: "app-store",
         output_directory: "./build",
         output_name: "<app_name>.ipa",
         xcargs: "DEVELOPMENT_TEAM=<team_id>",
       )
     end

     desc "TestFlight にアップロード"
     lane :beta do |options|
       build_release(build_number: options[:build_number])
       upload_to_testflight(skip_waiting_for_build_processing: true)
     end

     desc "Firebase App Distribution に配信"
     lane :distribute_firebase do |options|
       build_release(build_number: options[:build_number])
       firebase_app_distribution(
         app: ENV["FIREBASE_APP_ID"],
         groups: options[:groups] || "testers",
         release_notes: options[:release_notes] || "",
       )
     end

     desc "App Store に提出"
     lane :release do |options|
       fetch_api_key
       deliver(
         submit_for_review: options[:submit_for_review] || false,
         automatic_release: options[:automatic_release] || false,
         force: true,
         skip_binary_upload: true,
         skip_screenshots: true,
         precheck_include_in_app_purchases: false,
       )
     end

     desc "新しいデバイスを登録"
     lane :add_device do |options|
       device_name = options[:name] || prompt(text: "デバイス名: ")
       device_udid = options[:udid] || prompt(text: "UDID: ")
       register_devices(devices: { device_name => device_udid })
       match(type: "development", force_for_new_devices: true)
     end
   end
   ```
8. 結果を報告する:
   - 作成したファイル一覧（Gemfile, Appfile, Fastfile）
   - 次のステップ: `/ios-init-match` で証明書管理をセットアップ

## エラー対応

コマンド実行でエラーが発生した場合は、`shared/TROUBLESHOOTING.md` を参照しつつ以下の手順で対応する:

1. エラー出力を分析し、**原因** を特定する
2. **複数の解決方法** を具体的なコマンド付きで提示する
3. ユーザーにどの方法で進めるか **選択を求める**
4. 選択された方法を実行する
5. それでも失敗した場合は、別の解決方法を提示して再度選択を求める

### このスキル固有のよくあるエラー

- **`gem install` 失敗** → A) `rbenv version` で Ruby 環境を確認 B) `sudo` 権限の問題なら `rbenv exec gem install bundler` を使用
- **`.xcodeproj` が見つからない** → A) `find . -name "*.xcodeproj" -maxdepth 5` で検索範囲を拡大 B) ユーザーにプロジェクトパスを手動で指定してもらう
- **`bundle install` 失敗** → A) `Gemfile` の構文を確認 B) `rm -rf vendor/bundle && bundle install` でキャッシュクリア後に再実行

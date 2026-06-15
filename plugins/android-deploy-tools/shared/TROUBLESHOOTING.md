# 共通トラブルシューティングガイド（Android Deploy Tools）

## 共通エラー対応ルール（全スキル共通の行動指針）

- コマンドが失敗したら、エラー出力を分析し **原因** と **複数の解決方法** をユーザーに提示する
- 解決方法は具体的なコマンド付きで提示し、ユーザーに選択を求める
- ユーザーが選んだ方法を実行し、それも失敗したら次の対応策を提示する（繰り返し）
- 自分だけでは判断できない場合は正直にその旨を伝える

## よくあるエラーパターン集

### Firebase 認証（サービスアカウント）

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "Could not load the default credentials" / 認証失敗 | `GOOGLE_APPLICATION_CREDENTIALS` 未設定・パス不正 | A) サービスアカウント JSON 鍵のパスを環境変数に設定 B) 鍵ファイルの存在・読み取り権限を確認 |
| "The caller does not have permission" | サービスアカウントの権限不足 | Google Cloud コンソールでサービスアカウントに「Firebase App Distribution Admin」ロールを付与 |
| `FIREBASE_TOKEN` 利用時の deprecation 警告 | `firebase login:ci` トークンは非推奨 | サービスアカウント + `GOOGLE_APPLICATION_CREDENTIALS` 方式へ移行する手順を案内 |

### Firebase App Distribution

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "FIREBASE_APP_ID is not set" / App not found | App ID 未設定・不正 | Firebase Console (プロジェクト設定 → マイアプリ) から Android アプリの App ID（`1:xxxx:android:xxxx`）を確認・設定 |
| AAB 配信が拒否される | App Distribution が Google Play 未連携 | A) Firebase Console で Google Play 連携を有効化 B) APK 成果物にフォールバック |
| "Group ... does not exist" | テスターグループ未作成 | Firebase Console (App Distribution → テスターとグループ) でグループを作成 |

### ビルド（ネイティブ Android / Gradle）

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| `gradlew: Permission denied` | 実行権限なし | A) `chmod +x gradlew` B) CI 用に `git update-index --chmod=+x gradlew` でコミット |
| "SDK location not found" | `ANDROID_HOME` / `local.properties` 未設定 | A) `ANDROID_HOME` を設定 B) `local.properties` に `sdk.dir` を記載 |
| 署名エラー（release ビルド） | リリース署名設定の不足 | `signingConfigs` / keystore の設定状況を確認・案内 |
| Gradle ビルドタイムアウト | ビルド時間超過 | A) タイムアウト値を延長 B) `./gradlew clean` 後にクリーンビルド |

### ビルド（Flutter）

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| "flutter: command not found" | Flutter 未インストール / PATH 未設定 | A) Flutter SDK のインストール確認 B) CI では `subosito/flutter-action` を使用 |
| "No pubspec.yaml file found" | 実行ディレクトリが不正 | Flutter プロジェクトルート（`pubspec.yaml` のある場所）で実行 |
| `flutter build appbundle` 失敗 | 依存・署名・Gradle 設定の問題 | A) `flutter pub get` 後に再実行 B) `android/` の署名設定を確認 |

### CI (GitHub Actions)

| エラーパターン | 原因 | 解決方法（ユーザーに選択肢提示） |
|-------------|------|------|
| ワークフロー構文エラー | YAML の書式不正 | `actionlint` や YAML バリデーターで構文チェック |
| Secrets 未設定による実行失敗 | `FIREBASE_APP_ID` / `FIREBASE_SERVICE_ACCOUNT` 未登録 | `gh secret list` で設定状況を確認し、設定チェックリストを再表示 |
| サービスアカウント JSON が壊れる | Secret の改行・エスケープ問題 | JSON 全文をそのまま Secret に登録し、ワークフローで `printf '%s'` でファイル化しているか確認 |
| ワークフロー実行失敗 | 各種原因 | `gh run view <run_id> --log-failed` でログ確認手順を案内 |

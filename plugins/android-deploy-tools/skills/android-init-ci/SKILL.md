---
name: android-init-ci
description: Android / Flutter の Firebase App Distribution 配信を行う GitHub Actions ワークフローを構築する
disable-model-invocation: true
---

Android / Flutter アプリを Firebase App Distribution に配信する GitHub Actions ワークフローを構築する。サービスアカウント認証・AAB 配信を基本とする。

## 手順

1. プロジェクト種別を判定する:
   - `pubspec.yaml` が存在 → **Flutter プロジェクト**
   - `settings.gradle` / `settings.gradle.kts` が存在し `pubspec.yaml` が無い → **ネイティブ Android プロジェクト**
   - 判定できない場合はユーザーに確認する。
2. `.github/workflows/` ディレクトリが存在するか確認する（なければ `mkdir -p .github/workflows` で作成）。
3. ユーザーに以下を確認する:
   - Android / Flutter プロジェクトのパス（リポジトリルートからの相対パス、例: `.`, `android/`, `app/`）
   - Java バージョン（ネイティブ Android の場合、例: `17`）
   - Flutter バージョン / チャンネル（Flutter の場合、例: `stable`）
   - build variant（flavor / build type）:
     - flavor が定義されているか確認する（ネイティブ: `app/build.gradle(.kts)` の `productFlavors` / `flavorDimensions`、Flutter: `android/app/build.gradle` の `productFlavors`）。
     - 定義されていれば既定の flavor を確認する（例: `prod`）。build type は通常 `release`。
   - 既定の成果物（`aab` / `apk`、デフォルト: `aab`）
   - 既定のテスターグループ（デフォルト: `testers`）
4. `.github/workflows/deploy-android-firebase.yml` を作成する:
   - トリガー: `workflow_dispatch`（`groups`, `release_notes`, `artifact`, `flavor` の入力）
   - ランナー: `ubuntu-latest`
   - Secrets: `FIREBASE_APP_ID`, `FIREBASE_SERVICE_ACCOUNT`
   - ステップ（プロジェクト種別に応じて選択）:
     - checkout
     - **Flutter**: `subosito/flutter-action` で Flutter セットアップ → `flutter build appbundle --release`（APK の場合 `flutter build apk --release`）
     - **ネイティブ Android**: `actions/setup-java` で Java セットアップ → `./gradlew bundleRelease`（APK の場合 `./gradlew assembleRelease`）
     - サービスアカウント JSON をファイルに書き出し、`GOOGLE_APPLICATION_CREDENTIALS` に設定
     - `firebase-tools` をインストールして `firebase appdistribution:distribute` を実行

   テンプレート（Flutter の例。ネイティブの場合はビルドステップを Gradle に差し替える）:
   ```yaml
   name: Deploy Android to Firebase App Distribution

   on:
     workflow_dispatch:
       inputs:
         groups:
           description: "テスターグループ"
           required: false
           default: "testers"
         release_notes:
           description: "リリースノート"
           required: false
           default: ""
         artifact:
           description: "成果物 (aab / apk)"
           required: false
           default: "aab"
         flavor:
           description: "flavor（任意。未設定可）"
           required: false
           default: ""

   jobs:
     distribute:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4

         - uses: subosito/flutter-action@v2
           with:
             channel: stable

         - name: Build & locate artifact
           run: |
             FLAVOR_ARG=""
             [ -n "${{ inputs.flavor }}" ] && FLAVOR_ARG="--flavor ${{ inputs.flavor }}"
             if [ "${{ inputs.artifact }}" = "apk" ]; then
               flutter build apk --release $FLAVOR_ARG
               ARTIFACT_PATH=$(find build/app/outputs/flutter-apk -name '*.apk' | head -1)
             else
               flutter build appbundle --release $FLAVOR_ARG
               ARTIFACT_PATH=$(find build/app/outputs/bundle -name '*.aab' | head -1)
             fi
             echo "ARTIFACT_PATH=$ARTIFACT_PATH" >> "$GITHUB_ENV"

         - name: Write service account key
           run: |
             printf '%s' '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}' > "$RUNNER_TEMP/firebase-sa.json"

         - name: Distribute to Firebase App Distribution
           env:
             GOOGLE_APPLICATION_CREDENTIALS: ${{ runner.temp }}/firebase-sa.json
             FIREBASE_APP_ID: ${{ secrets.FIREBASE_APP_ID }}
           run: |
             npm install -g firebase-tools
             firebase appdistribution:distribute "$ARTIFACT_PATH" \
               --app "$FIREBASE_APP_ID" \
               --groups "${{ inputs.groups }}" \
               --release-notes "${{ inputs.release_notes }}"
   ```

   ネイティブ Android のビルドステップ例（上記の Flutter セットアップ + Build ステップと差し替える）:
   ```yaml
         - uses: actions/setup-java@v4
           with:
             distribution: temurin
             java-version: "17"

         - name: Build & locate artifact
           run: |
             if [ -n "${{ inputs.flavor }}" ]; then
               VARIANT="$(echo '${{ inputs.flavor }}' | sed 's/^./\U&/')Release"
             else
               VARIANT="Release"
             fi
             if [ "${{ inputs.artifact }}" = "apk" ]; then
               ./gradlew assemble${VARIANT}
               ARTIFACT_PATH=$(find app/build/outputs/apk -name '*.apk' | head -1)
             else
               ./gradlew bundle${VARIANT}
               ARTIFACT_PATH=$(find app/build/outputs/bundle -name '*.aab' | head -1)
             fi
             echo "ARTIFACT_PATH=$ARTIFACT_PATH" >> "$GITHUB_ENV"
   ```
   - `<Variant>` は flavor + build type をキャメルケースで結合し先頭を大文字にしたもの（例: flavor `prod` → `bundleProdRelease`、flavor 無し → `bundleRelease`）。flavor が複数次元の場合は `./gradlew app:tasks --group=build` で正確なタスク名を確認する。
5. 結果を報告する:
   - 作成したワークフローファイル
   - 設定が必要な GitHub Secrets 一覧:

     | Secret | 用途 |
     |--------|------|
     | `FIREBASE_APP_ID` | Firebase App ID（`1:xxxx:android:xxxx`） |
     | `FIREBASE_SERVICE_ACCOUNT` | サービスアカウント JSON 鍵の全文（「Firebase App Distribution Admin」ロール付与） |

   - 手動実行方法:
     ```bash
     gh workflow run deploy-android-firebase.yml \
       -f groups="testers" -f release_notes="..." -f artifact="aab" -f flavor="prod"
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
- **Secrets 未設定による実行失敗** → `gh secret list` で設定状況を確認し、`FIREBASE_APP_ID` / `FIREBASE_SERVICE_ACCOUNT` の設定チェックリストを再表示
- **`gradlew: Permission denied`** → `git update-index --chmod=+x gradlew` で実行権限を付与してコミット
- **ワークフロー実行失敗** → `gh run view <run_id> --log-failed` でエラーログを確認し、原因に応じた解決策を提示

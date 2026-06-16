---
name: android-deploy-firebase
description: Android / Flutter アプリを Firebase App Distribution で配信する
disable-model-invocation: true
---

Android / Flutter アプリを Firebase App Distribution で配信する。GitHub Actions またはローカルで Firebase CLI を使ってビルド・配信を行う。AAB を基本とし、Google Play 未連携の場合は APK にフォールバックする。認証はサービスアカウント（`GOOGLE_APPLICATION_CREDENTIALS`）を使う（`FIREBASE_TOKEN` は非推奨のため使わない）。

## 手順

1. 現在のブランチを確認する。配信元のブランチを確認（main 以外からの配信も許可）。
2. `git fetch origin && git pull origin <ブランチ名>` でローカルを最新化する。
3. プロジェクト種別を判定する:
   - `pubspec.yaml` が存在 → **Flutter プロジェクト**
   - `settings.gradle` / `settings.gradle.kts` が存在し `pubspec.yaml` が無い → **ネイティブ Android プロジェクト**
   - 判定できない場合はユーザーに確認する。
4. ビルド variant（flavor / build type）を確認する:
   - flavor が定義されているか検出する:
     - ネイティブ Android: `app/build.gradle` / `app/build.gradle.kts` の `productFlavors` / `flavorDimensions` を確認する。`./gradlew app:tasks --group=build` でも `bundle<Flavor>Release` 形式のタスク一覧を確認できる。
     - Flutter: `android/app/build.gradle` の `productFlavors`、または `flutter build` で使う `--flavor` を確認する。
   - flavor が存在する場合はどれを配信するかユーザーに確認する（例: `prod`, `staging`）。build type は通常 `release`。
   - flavor が無い場合は variant = build type（`release`）。
5. ユーザーに以下を確認する:
   - テスターグループ名（デフォルト: `testers`）
   - リリースノート（空欄可）
   - 成果物の種類（デフォルト: AAB。Google Play 未連携の場合は APK）
6. デプロイ方法を選択する:
   - **GitHub Actions（推奨）**:
     ```bash
     gh workflow run deploy-android-firebase.yml \
       -f groups="<groups>" \
       -f release_notes="<notes>" \
       -f artifact="aab" \
       -f flavor="<flavor>"
     ```
     - flavor が無い場合は `-f flavor=""` を渡す。
     - `gh run list --workflow=deploy-android-firebase.yml --limit=1` でワークフロー起動を確認する。
     - ワークフローが存在しない場合は `/android-init-ci` で先に作成するよう案内する。
   - **ローカル実行**:
     1. variant 付きのタスクでビルドする（ビルドには時間がかかるため、タイムアウトを長めに設定する（10分））。
        ここで `<Variant>` は flavor + build type をキャメルケースで結合し先頭を大文字にしたもの（例: flavor `prod` + `release` → `ProdRelease`、flavor 無し → `Release`）。

        | プロジェクト | 成果物 | ビルドコマンド |
        |------------|-------|--------------|
        | ネイティブ Android | AAB | `./gradlew bundle<Variant>`（flavor 無し: `./gradlew bundleRelease`） |
        | ネイティブ Android | APK | `./gradlew assemble<Variant>`（flavor 無し: `./gradlew assembleRelease`） |
        | Flutter | AAB | `flutter build appbundle --release --flavor <flavor>`（flavor 無し: `--flavor` を省略） |
        | Flutter | APK | `flutter build apk --release --flavor <flavor>`（flavor 無し: `--flavor` を省略） |

     2. 生成された成果物のパスを `find` で特定する（出力パスは variant により変わるため決め打ちしない。複数ヒットする場合は variant 名を含むディレクトリのものを選ぶ）:
        ```bash
        # ネイティブ Android: AAB は app/build/outputs/bundle/<variant>/、APK は app/build/outputs/apk/<flavor>/<buildType>/
        find app/build/outputs/bundle -name '*.aab'   # AAB
        find app/build/outputs/apk    -name '*.apk'   # APK
        # Flutter
        find build/app/outputs/bundle      -name '*.aab'   # AAB
        find build/app/outputs/flutter-apk -name '*.apk'   # APK
        ```
     3. 配信する:
        ```bash
        firebase appdistribution:distribute "<特定した成果物パス>" \
          --app "$FIREBASE_APP_ID" \
          --groups "<groups>" \
          --release-notes "<notes>"
        ```
     - `FIREBASE_APP_ID` 環境変数（`1:xxxx:android:xxxx` 形式）が設定されていることを確認する。
     - サービスアカウント認証を確認する: `GOOGLE_APPLICATION_CREDENTIALS` にサービスアカウント JSON 鍵のパスが設定されていること。
7. 結果を報告する:
   - ビルドした variant（flavor / build type）
   - 配信した成果物（AAB / APK）とパス
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

- **`FIREBASE_APP_ID` 未設定** → Firebase Console (プロジェクト設定 → マイアプリ) から Android アプリの App ID（`1:xxxx:android:xxxx`）を確認し、環境変数に設定する手順を案内
- **AAB 配信が拒否される（Google Play 未連携）** → A) Firebase Console (App Distribution → 設定) で Google Play との連携を有効化 B) APK 成果物（`./gradlew assembleRelease` / `flutter build apk --release`）にフォールバックして再配信
- **`GOOGLE_APPLICATION_CREDENTIALS` 未設定 / 認証エラー** → Google Cloud コンソールでサービスアカウントに「Firebase App Distribution Admin」ロールを付与し、JSON 鍵を発行して環境変数にパスを設定する手順を案内（`FIREBASE_TOKEN` は非推奨のため使わない）
- **テスターグループが存在しない** → Firebase Console (App Distribution → テスターとグループ) でグループを作成する手順を案内
- **`Task 'bundleRelease' not found` / 成果物が見つからない** → flavor が定義されているため variant 名がタスク名・出力パスに含まれている。`./gradlew app:tasks --group=build` で正しい `bundle<Variant>` / `assemble<Variant>` タスク名を確認し、成果物は `find` で再特定する

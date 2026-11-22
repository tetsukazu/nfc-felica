# AGENTS.md — 共通AI開発ルール
<!-- common-rules revision: 2026-11-22.1 -->
適用対象: 管理対象のAI・開発者。共通ルールの暫定正本は tetsukazu/common の AGENTS.md。各リポジトリへ同一内容を同期する。固有ルールは末尾または既存文書に保持する。既存仕様と衝突する場合は移行課題として記録し、動作・契約を勝手に変更しない。

## 設計・開発
- 人間は要求・重要な意思決定・受入確認、AIは要求整理、設計、実装、自動テスト、文書、PR、CI、配布確認を担当する。最も単純な必要十分の構成を優先し、不要な依存やリファクタリングを避ける。
- 着手前に README、既存指示書、docs、関連Issueを確認。Issueに目的、範囲、受入条件、テスト観点、優先順位、依存を記録し、重複を避ける。小さな論理単位に分割する。
- 日本語名・英語名を定義。新規リポジトリ名は小文字の英数字を基本とする。複数コンポーネントはモノレポを基本とし apps/android、apps/web、apps/server、apps/windows 等を必要に応じて使う。既存構成を無断で移動しない。
- Android applicationId と基本パッケージは link.iwa.<appname>。既存の applicationId、署名鍵は維持する。
- README.md、AGENTS.md、CHANGELOG.md、docs/、重要な判断は docs/adr/。仕様の正本は最新版の設計書、過去はGit履歴。仕様変更はコードと文書を同一PRで更新。CLAUDE.md 等は共通ルールを参照し固有指示を保持する。

## Git・マージ
- main（既存の master 等のデフォルトブランチを含む）への直接pushは禁止。作業ブランチとPRを必須とする。1コミット1目的、Conventional Commits（feat(android): 検索機能を追加する 等）、原則日本語。強制push・履歴改変・破壊的resetは禁止（明示承認時のみ）。
- Squash Merge を標準とし、マージ後は作業ブランチを削除する。必須CI成功、最新ベースとの競合なし、受入条件、テスト、仕様書、採番、秘密情報、必要なレビューを確認してAIが自動マージしてよい。必要な承認や保護を迂回しない。
- データ削除・破壊的移行、認証認可・暗号・署名鍵の重大変更、外部送信拡大、継続費用発生、互換性破壊は事前承認必須。CI未実行・失敗・未確認は成功扱いしない。テストを無効化して通さない。
- PRには目的、変更、関連Issue、仕様・セキュリティ・データへの影響、テスト結果、バージョン、リリースノート、受入確認を記載。

## バージョン・リリース
- アプリ別に MAJOR.MINOR.PATCH。互換性破壊はMAJOR、後方互換の機能追加・大幅改善はMINOR、修正・軽微改善・性能改善はPATCH。複数変更は最大区分。配布する内部変更もPATCH。文書/AI指示のみや配布不要なCI設定変更は採番しない。
- 変更したアプリだけ、1 PR につき最終段階で1回採番。Android versionName は上記、versionCode は単調増加・原則+1（計算式にしない）。配布済み番号を異なる成果物に再利用しない。Web/Server は package.json 等の標準設定とロックファイルを同期。既存番号を継承。
- マージ直前に最新ベースで再採番しCI再実行、同一アプリのマージ・配布を直列化する。配布失敗時は同一成果物の再実行と修正後の新規リリースを区別。
- アプリ内の設定/情報画面、Webのフッター等でバージョン表示、Serverはバージョン確認API等。可能なら該当リリースノートへの導線（非公開の場合は閲覧可能な方法）。
- 標準はPRマージ時に自動リリース、タグ pr-<PR番号>、PR本文の ## リリースノート が唯一の生成元。CHANGELOG.mdへ履歴を蓄積。日本語・利用者目線で概要、新機能、改善、修正、既知の問題、テスター確認、必要な操作（再ログイン等は冒頭）を記述。空節・内部実装・秘密値は除外。
- GitHub Releases と Android の Firebase App Distribution の両方に同一アプリ・同一バージョンのノートを自動反映する。モノレポは対象アプリ別の節を抽出。欠落・空のノートは配布前に失敗させる。WindowsはGitHub Releases、Web/Serverは既存の指定環境。Androidは既存鍵で署名済みAPK。変更対象外は配布しない。
- 片側のみ成功したら部分成功として報告し、同一成果物の再実行で重複・巻戻しを防ぐ。既存のタグ起点方式は動作確認を伴う移行PRで切替し、それまでは既存の方式を保持。

## CI/CD
- 正式ビルド、静的解析、テスト、パッケージ化、配布・デプロイは GitHub Actions、デフォルトは **Self-hosted Runner**。RUNNER_LABELS（JSON配列）で指定。例: `runs-on: ${{ fromJSON(vars.RUNNER_LABELS || '["self-hosted","windows","x64"]') }}`（Windows対応ジョブの場合）。Linux/OS別ジョブは実際の要件に合わせる。GitHub-hostedは明示的指定のみ、自動フォールバック禁止。
- SDK/JDK・キャッシュ再利用、ジョブ別チェックアウト・作業領域、署名鍵・認証情報・一時ファイルの後処理、信頼できないPRの隔離。OS/シェル/Docker/ツールを確認して移行する。ビルド・テスト・署名・配布の時間を記録。変更対象および依存に絞る。軽量ローカル検証は固有ルールの範囲で可、正式ビルド/デプロイをAIセッションから直接行わない。

## 設定名・秘密情報
- GitHub Secrets、Variables、環境変数は UPPER_SNAKE_CASE。同一用途は同一名称、アプリ名接頭辞は原則不要。必要なものだけ使用し、設定ファイルの正本と二重管理しない。名称、型、用途、必須/任意、設定先、既定値を文書化（.env.example に秘密値を入れない）。
- Secrets/署名鍵の値はアプリ・リポジトリ別に個別管理。Organization全体の共通化はしない。既存WIF/Secret Manager等を維持し、不必要なサービスアカウント鍵を発行しない。ログ・Git・Issue・PR・ノートへの秘密情報出力禁止。google-services.jsonの扱いは既存方針。
- 標準Secrets: ANDROID_KEYSTORE_BASE64, ANDROID_KEYSTORE_PASSWORD, ANDROID_KEY_ALIAS, ANDROID_KEY_PASSWORD, ANDROID_DEBUG_KEYSTORE_BASE64, GOOGLE_SERVICES_JSON_BASE64（保管が必要な場合）, FIREBASE_SERVICE_ACCOUNT_JSON（鍵認証時のみ）, GOOGLE_CLIENT_SECRET（機密クライアントのみ）, GOOGLE_MAPS_API_KEY（秘密として管理する場合）, MONGODB_URI, MONGODB_TEST_URI, R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY, CLOUDFLARE_API_TOKEN。
- 標準Variables: RUNNER_LABELS, JAVA_VERSION, NODE_VERSION, FLUTTER_VERSION, PYTHON_VERSION, FIREBASE_TOOLS_VERSION（既存のバージョン定義優先）、FIREBASE_PROJECT_ID, FIREBASE_ANDROID_APP_ID, FIREBASE_WEB_APP_ID, FIREBASE_API_KEY（公開用）, FIREBASE_AUTH_DOMAIN, FIREBASE_STORAGE_BUCKET, FIREBASE_MESSAGING_SENDER_ID, FIREBASE_TESTER_GROUPS, FIREBASE_HOSTING_SITE, GCP_PROJECT_ID, GCP_REGION, GCP_WIF_PROVIDER, GCP_DEPLOY_SERVICE_ACCOUNT, GCP_RUNTIME_SERVICE_ACCOUNT, GCP_ARTIFACT_REGISTRY_REPOSITORY, CLOUD_RUN_SERVICE, APP_NAME, APP_NAME_JA, ANDROID_APPLICATION_ID, API_BASE_URL, STORAGE_API_URL, GOOGLE_WEB_CLIENT_ID, MONGODB_DATABASE, CLOUDFLARE_ACCOUNT_ID, R2_BUCKET_NAME。公開用フロントエンド変数には必要なフレームワーク接頭辞（VITE_等）を付けてマッピング。秘密値を公開ビルドに入れない。
- 未設定の必須変数は名前と理由のみ報告。移行は新名称設定→参照変更（必要なら旧名称フォールバック）→CI/配布確認→旧名称廃止。Secretsの登録済み値は読み出せないため、所有者による再登録が必要な場合は手順を報告。既存環境・認証を壊さない。

## 命名・DB
- Kotlin/TS/Dartの変数・関数は camelCase、Pythonは snake_case、C#公開メンバーはPascalCase、クラスはPascalCase。新規MongoDB/SQLite項目は snake_case、APIパスおよび通常の新規Markdownファイルは kebab-case。言語標準・既存契約優先。
- 同じ意味は同じ名前: created_at, updated_at, deleted_at, recorded_at, captured_at, started_at, ended_at, synced_at, user_id, owner_id, schema_version, format_version, latitude, longitude, distance_m, duration_seconds, weight_kg, energy_kcal。時刻はUTC、表示は利用者のTZ。日付のみは区別し、型・単位を明記する。
- MongoDB既存定義は https://github.com/tetsukazu/datacatalog を参照し、既存フィールド・列挙型・型を維持する。変更時はカタログ更新要否を確認。権限不足なら依頼を報告。クライアントの直接DB接続禁止（サーバAPI経由）。本番DB変更は無断で行わない。

## セキュリティ・テスト
- 認証・認可を区別し、サーバ側で本人・権限を検証、最小権限、必要最小限の外部送信・ログ。実資格情報・個人情報をテストに使わない。既存固有の厳しいルールを維持。
- AIは正常/異常/境界、入力、認証認可、同時実行・二重登録、整合性、切断・リトライ・オフライン、消失・破損、バックアップ復元、移行、回帰、性能の適用項目をテスト。修正には可能な限り再発防止テスト。実機/人間による受入観点（目的、操作、期待結果、環境、優先度）を生成する。CI成功を受入済みとみなさない。
- 完了報告は作業内容、変更ファイル、Issue/PR、CI・テスト、マージ、版、配布、未完了と受入事項。未確認は明記。

## 適用・例外
- 文書専用・旧式・単一用途リポジトリでは該当事項のみ適用。Android/Firebase/配布等を不要に追加しない。既存デフォルトブランチ名、プロジェクト、実装、署名鍵を維持。
- 現行ワークフローと規定が異なる場合は移行課題を記録し、実機/Runner/権限/配布確認後に切替。指示書の変更だけで実装済みとみなさない。固有ルールを保存し、矛盾・例外は理由、対象、解消条件を明示する。
<!-- end common-rules -->

## 固有ルール・移行課題
既存NFC/FeliCaの機器・SDK・実機要件を維持。継続開発時はRunnerの機器依存と実機確認を文書化。既存ID・配布を変更しない。
既存のREADMEと文書を参照し、現行動作を維持する。

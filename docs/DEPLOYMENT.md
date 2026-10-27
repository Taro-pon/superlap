# デプロイ・運用フロー

GitHubとVercelによる公開フロー、ドメイン、更新方法、ロールバック手順をまとめたドキュメントです。

**重要**: このリポジトリ内には Vercel の設定ファイル（`vercel.json` 等）は存在せず、Vercelダッシュボード側の設定内容（プロジェクト設定、環境変数、ドメインのDNS設定など）はこのドキュメント作成時点で確認できていません。以下、ソースコード・Git履歴から確認できた事実と、確認できなかった事項（未確認）を明確に分けて記載します。

## 1. 確認できている事実

- GitHubリモートリポジトリ: `https://github.com/Taro-pon/superlap.git`（`origin`、ブランチ `main`）
- コミット履歴に `1105db5 Test auto deploy`（2026-06-29）があり、GitHubへのプッシュに連動した自動デプロイのテストが行われた記録がある → **GitHub連携によるVercelの自動デプロイが過去に設定・動作確認されている**ことが示唆される（Vercelダッシュボード側の現在の設定自体は未確認）。
- `index.html` のOGPタグに本番URLとして `https://www.superlap.net/` が記載されている → **独自ドメイン `www.superlap.net` が本番URLとして使用されている**ことが確認できる。
- `index.html` に Google Search Console 用の `google-site-verification` メタタグが設置されている。Google Search Console管理画面にて、対象URL `https://www.superlap.net/` の所有権確認・HTTPS・トップページのインデックス登録が確認済み（2026-10-27時点、詳細は[docs/ARCHITECTURE.md](ARCHITECTURE.md)参照）。サイトマップの登録状態・プロパティ種別は未確認。
- リポジトリに `vercel.json` や `now.json` などのVercel設定ファイルは存在しない → ビルド設定はVercel側のデフォルト（静的ファイルとしての配信）、またはVercelダッシュボード上のプロジェクト設定に依存していると推測される（詳細未確認）。
- リポジトリルートに `sitemap.xml`・`robots.txt` を追加済み（2026-10-27）。静的ファイルのため、`main` へのデプロイ後は `https://www.superlap.net/sitemap.xml` および `https://www.superlap.net/robots.txt` としてそのまま公開される見込み（デプロイ後の実際の到達確認は未実施）。

## 2. 未確認の事項（Vercelダッシュボード側で確認が必要）

以下はこのリポジトリのファイルからは確認できず、Vercelダッシュボードにログインしての確認が必要です。

- Vercelプロジェクト名、プロジェクトの作成者・所属チーム
- Production Branch の設定（`main` を本番ブランチとしているかどうか）
- Framework Preset / Build Command / Output Directory の設定内容
- 環境変数の設定有無
- 独自ドメイン `www.superlap.net`（および `superlap.net`）のDNS設定・ネームサーバー・リダイレクト設定
- Preview Deployments（PRごとのプレビュー環境）の有効化状況
- デプロイ通知（Slack連携等）の設定
- デプロイ後の `sitemap.xml` / `robots.txt` の実際の到達確認、およびGoogle Search Consoleへのサイトマップ登録（URL送信）の実施状況

## 3. 公開フロー（想定される標準的なVercel + GitHub連携の挙動）

GitHub連携によるVercelのデプロイは、一般的に以下のフローで動作します（Vercelの標準的な仕組みに基づく説明であり、本プロジェクト固有の設定値は未確認です）。

1. `main` ブランチに変更がプッシュされると、Vercelが自動検知してビルド・デプロイを実行する
2. デプロイが成功すると、本番URL（本プロジェクトの場合 `https://www.superlap.net/` および Vercel発行のデフォルトURL）に反映される
3. `main` 以外のブランチへのプッシュや、GitHub上でのPull Request作成時には、Vercelが個別のプレビューURLを発行する（Preview Deploymentsが有効な場合）

実際にこの挙動どおりに設定されているかは、Vercelダッシュボードでの確認が必要です。

## 4. 更新方法

本サイトはビルド処理を持たない静的HTMLのため、更新作業は以下の手順になります。

1. `index.html` / `privacy.html` 等を編集する
2. ローカルでブラウザ（またはローカルHTTPサーバー）で表示を確認する（PC・スマートフォン両方のレイアウトを確認。手順は[README.md](../README.md)参照）
3. 変更内容を `git add` → `git commit` する
4. `git push origin main` で GitHub にプッシュする（**GitHubへのプッシュはユーザーの指示があった場合のみ実施** — [CLAUDE.md](../CLAUDE.md)参照）
5. Vercel側で自動デプロイが実行される（上記3章参照）。デプロイ完了は Vercel ダッシュボードまたは GitHub のコミットステータスで確認する

## 5. ロールバック手順

リポジトリ内に専用のロールバックスクリプト等は存在しません。想定される手段は以下の2系統です。

### Vercelダッシュボードから戻す（推奨・最速）

Vercelは各デプロイの履歴を保持しており、ダッシュボードの「Deployments」一覧から過去の正常なデプロイを選択し「Promote to Production」（または同等の操作）を行うことで、ビルドをやり直さずに即座に本番環境を過去の状態に戻せます。この操作はVercel側の機能であり、Git履歴を変更しません。

### Gitで戻してから再デプロイする

1. 問題のあるコミットを特定する（`git log` で確認）
2. 直前の正常な状態に戻す新しいコミットを作成する（`git revert <コミットハッシュ>` を推奨。`git reset --hard` 等の履歴を破壊する操作は、必要性と影響範囲をユーザーと確認した上でのみ実施する）
3. `main` にプッシュし、Vercelの自動デプロイで反映させる

## 6. 本番反映前の確認

[CLAUDE.md](../CLAUDE.md)にも記載の通り、本番環境（`main` ブランチへのプッシュ）に反映する前に、必ずユーザーへ変更内容を確認してください。

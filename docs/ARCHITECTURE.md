# アーキテクチャ

Superlapコーポレートサイトの構造・主要機能・使用技術・外部サービス連携をまとめたドキュメントです。記載内容はソースコード・設定ファイルを直接確認した事実のみとし、確認できなかった事項は「未確認」と明記しています。

## 1. 全体方針

- 静的サイト（HTML / CSS / JavaScript のみ）。ビルドツール、パッケージマネージャ、フレームワークは一切使用していない（`package.json` 等のビルド関連ファイルはリポジトリ内に存在しない）。
- CSSはすべて `index.html` / `privacy.html` 内の `<style>` タグにインライン記述（外部CSSファイルなし）。
- JavaScriptも `index.html` 末尾の `<script>` タグ内にインライン記述（外部JSファイルなし）。
- フォントは Google Fonts を `@import` で読み込み（Noto Sans JP, Inter）。

## 2. ページ構成とナビゲーション

Gitコミット履歴から、過去に複数ページ構成を試みた後（`1fc1b9d Split single-page site into multiple pages`）、シングルページ構成に戻した（`3ef3cb1 Revert to single-page layout with anchor navigation`）経緯が確認できます。

| ページ | ファイル | 役割 |
|---|---|---|
| トップページ | `index.html` | シングルページ構成。アンカーリンクで各セクションに移動 |
| プライバシーポリシー | `privacy.html` | 独立ページ。`index.html` と同じヘッダー/CSSを個別に保持 |

### index.html のセクション構成

1. `nav`（固定ヘッダー・ロゴ・ナビゲーション・ハンバーガーメニュー）
2. `.hero`（ヒーローセクション、会社のキャッチコピー）
3. `#philosophy`（理念：Mission / Vision / Value の3項目）
4. `#services`（運営サービス：主力事業「Anymany」＋過去プロジェクト3件）
5. `#company`（会社概要：会社名・代表者・設立・所在地・事業内容・主要設備を表形式で記載）
6. `#contact`（お問い合わせフォーム）
7. `footer`（ロゴ・著作権表示・プライバシーポリシーへのリンク）

ナビゲーション（PC: `.nav-links`、モバイル: `.mobile-menu`）はいずれも `#philosophy` `#services` `#company` `#contact` へのアンカーリンクで構成されています。

`privacy.html` のナビゲーションは `index.html#philosophy` のように、トップページへの絶対アンカーリンクになっています。

### 運営サービスセクションの内容

- 主力事業として「Anymany」（オンラインレーザー加工サービス）を大きく掲載し、外部リンク（`https://anymany.net`、`target="_blank"`）を設置。
- 過去プロジェクトとして3件をカード形式で掲載：
  - FabLab Hajata（`https://fablabhakata.com/` へのリンクあり）
  - Touch Design Works（`https://www.touch-design.net/` へのリンクあり）
  - Our Soup（食べるスープ研究所。外部リンクなし、ロゴと説明文のみ）

## 3. 画像・SVG・ファビコンの管理方法

- ロゴ類はすべてSVGとして `assets/logo/` 配下にファイルで保存されており、`index.html` 内では `<img>` ではなく `<svg>` をインラインで直接埋め込む形で使用（`superlap` ロゴ、`Anymany` ロゴ、Our Soup ロゴなど）。
- `assets/logo/` 配下のファイルのうち、`fablabhajata_logo.svg` と `touch_design_works_logo.svg` は index.html 内に該当する `<img>`/インラインSVGの直接参照は確認できず（ファイルとして存在するが使用箇所未確認）。
- `assets/images/` ディレクトリは存在するが、調査時点では空（ファイルなし）。
- ファビコン関連ファイル（`favicon.ico` / `favicon-32x32.png` / `apple-touch-icon.png`）はリポジトリのルート直下と `assets/logo/` の両方に存在し、内容は同一（`diff` で確認済み・バイナリ差分なし）。`index.html` / `privacy.html` の `<head>` ではルート直下のパス（`/favicon.ico` 等、絶対パス）と、`assets/logo/superlap_favicon.svg`（相対パス、SVGアイコン）の両方を参照している。
- `.gitignore` では `.DS_Store` のみ除外対象。

## 4. レスポンシブ対応

- ブレークポイントは `@media (max-width: 768px)` の1箇所のみ（`index.html` / `privacy.html` 共通のCSS設計）。
- モバイル時の主な変更点：
  - PC用ナビゲーション（`.nav-links`）を非表示にし、ハンバーガーメニュー（`.hamburger` / `.mobile-menu`）を表示
  - `--gutter` カスタムプロパティを `2rem` → `1.25rem` に縮小
  - `.mvv-list`（理念の3カラム）、`.past-grid`（過去プロジェクトの3カラム）を1カラムに変更
  - `.company-layout`、`.contact-layout` を1カラムに変更、`position: sticky` を解除
  - フォームの2カラム行（`.form-row`）を1カラムに変更
  - フッターを縦積み・中央揃えに変更
- レイアウトの基準幅は `--max-width: 960px`、横幅調整は `--gutter` カスタムプロパティで一元管理（`aa608e4 Unify horizontal layout with --gutter variable` のコミットで導入）。

## 5. お問い合わせ機能

- `#contact` セクション内の `<form id="contactForm">` で構成。
- 送信先: [Formspree](https://formspree.io/)（`action="https://formspree.io/f/mykanbyj"`, `method="POST"`）。
- 入力項目: お名前（必須）、会社名・屋号、メールアドレス（必須）、電話番号、お問い合わせ内容（必須）。
- JavaScript（`index.html` 末尾の `<script>`）で `submit` イベントを `preventDefault()` し、`fetch()` によりFormspreeへ非同期送信。成功時はフォームをリセットし `#formSuccess` のメッセージを表示、送信ボタンを非表示化。失敗時はボタンのテキストを「送信に失敗しました。再度お試しください。」に変更して再送信可能にする。
- サーバーサイドの処理（バックエンドAPI等）は持たず、Formspree側の管理画面でのみ送信内容を確認する構成（Formspreeの詳細な管理設定・通知先メールアドレス等は未確認）。

## 6. SEO・OGP・構造化データ

`index.html` の `<head>` 内で以下を確認：

- `<title>` タグ、`<meta name="description">` を設定
- OGPタグ: `og:title` / `og:description` / `og:url`（`https://www.superlap.net/`） / `og:type`（`website`）
- JSON-LD構造化データ（`@type: Organization`）: 会社名、英語名、URL、住所（`PostalAddress`）、代表者（`founder`）、提供サービス（`hasOfferCatalog` として Anymany を記載）
- `og:image` は確認できず（**未確認／未設定の可能性**）。
- `privacy.html` には OGP・JSON-LD・`meta description` の記載は確認できず（`index.html` のみに実装されている）。
- `sitemap.xml` / `robots.txt` はリポジトリ内に確認できず（**未確認**）。

関連コミット: `271269b Add SEO meta tags, OGP, and JSON-LD structured data`

## 7. 外部サービス連携

### Google Search Console

- `index.html` の `<head>` に `<meta name="google-site-verification" content="...">` タグが設置されている（`b83205c` → `0299f4a` で修正済み）。
- 実際のGoogle Search Console上でのプロパティ設定・インデックス状況・サイトマップ登録状況は**未確認**（Search Console管理画面はこのリポジトリの範囲外）。

### Google Fonts

- `@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&family=Inter:wght@400;500;600;700&display=swap')` を `index.html` / `privacy.html` のCSS内で読み込み。

### Formspree

- お問い合わせフォームの送信先として使用（詳細は5章）。

### アクセス解析

- `privacy.html` の「5. アクセス解析」の章に解析ツール使用についての記載があるが、`index.html` 内に Google Analytics や他のアクセス解析タグ（`gtag`, `analytics.js` 等）の実装は**確認できず**。プライバシーポリシーの記載内容と実装が一致していない可能性がある（**要確認事項**、[CLAUDE.md](../CLAUDE.md)にも注記）。

## 8. 使用技術まとめ

| 分類 | 内容 |
|---|---|
| マークアップ | HTML5（素のHTML、テンプレートエンジンなし） |
| スタイル | 素のCSS（インライン `<style>`、CSS変数によるデザイントークン管理） |
| スクリプト | 素のJavaScript（ES6、外部ライブラリなし） |
| フォント | Google Fonts（Noto Sans JP, Inter） |
| フォーム送信 | Formspree |
| バージョン管理 | Git / GitHub（`Taro-pon/superlap`） |
| ホスティング | Vercel（詳細は[DEPLOYMENT.md](DEPLOYMENT.md)） |

## 9. GitHubとの連携状況

- リモートリポジトリ: `https://github.com/Taro-pon/superlap.git`（`origin`）
- ブランチ: `main` のみ運用（調査時点でローカルに他ブランチは存在しない）
- GitHub Actions等のCIワークフロー（`.github/workflows/`）はリポジトリ内に存在しない（**未確認／未導入**）

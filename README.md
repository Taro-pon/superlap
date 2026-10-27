# superlap

株式会社スーパーラップ（superlap, Inc.）のコーポレートサイトです。

公式URL: https://www.superlap.net/

## 概要

- 福岡市を拠点に、オンラインレーザー加工サービス「Anymany」を運営する会社のコーポレートサイト
- 構成はシングルページ（`index.html`）＋独立ページ（`privacy.html`）
- フレームワークやビルドツールは使用せず、静的HTML/CSS/JavaScriptのみで構成
- GitHub（`Taro-pon/superlap`）と連携し、Vercelでホスティング・公開

詳しい構成・技術・運用フローは以下のドキュメントを参照してください。

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — サイト構造・主要機能・使用技術
- [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) — 公開フロー・ドメイン・更新・ロールバック
- [docs/CHANGELOG.md](docs/CHANGELOG.md) — 変更履歴
- [CLAUDE.md](CLAUDE.md) — Claude Codeでの開発ルール

## ディレクトリ構成

```
superlap/
├── index.html                  # トップページ（理念・サービス・会社概要・お問い合わせ）
├── privacy.html                # プライバシーポリシー（独立ページ）
├── favicon.ico                 # ファビコン（ルート直下、index.html / privacy.html から参照）
├── favicon-32x32.png           # ファビコン 32x32（ルート直下）
├── apple-touch-icon.png        # Apple端末用アイコン（ルート直下）
├── assets/
│   ├── images/                 # 画像用ディレクトリ（調査時点では空）
│   └── logo/                   # 各サービス・会社のロゴSVG、ファビコン類
│       ├── superlap_logo.svg
│       ├── superlap_favicon.svg
│       ├── anymany_logo.svg
│       ├── fablabhajata_logo.svg
│       ├── touch_design_works_logo.svg
│       ├── oursoup_logo.svg
│       ├── favicon.ico              # ルート直下のものと内容は同一
│       ├── favicon-32x32.png        # ルート直下のものと内容は同一
│       └── apple-touch-icon.png     # ルート直下のものと内容は同一
├── .gitignore                  # .DS_Store を除外
└── README.md
```

※ `favicon.ico` / `favicon-32x32.png` / `apple-touch-icon.png` はルート直下と `assets/logo/` の両方に同一内容のファイルが存在しています（重複）。HTMLからは両方とも参照されています（詳細は[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)参照）。

## ローカルでの確認方法

ビルドツールを使用していないため、ブラウザで直接HTMLファイルを開くか、簡易HTTPサーバーで配信して確認します。

### ブラウザで直接開く

```bash
open index.html
```

### 簡易HTTPサーバーで確認する（推奨）

相対パスでの画像・SVG読み込みやフォームの挙動を本番に近い形で確認したい場合はローカルサーバー経由を推奨します。

```bash
# Python3 がインストールされている場合
python3 -m http.server 8000
# → http://localhost:8000 をブラウザで開く
```

```bash
# Node.js がインストールされている場合（npx経由、インストール不要）
npx serve .
```

PC表示とスマートフォン表示の両方を、ブラウザの開発者ツール（レスポンシブモード）で確認してください（ブレークポイントは `max-width: 768px`）。

## お問い合わせフォームについて

お問い合わせフォームの送信先は [Formspree](https://formspree.io/) です（`index.html` 内 `action="https://formspree.io/f/mykanbyj"`）。ローカル環境でも実際に送信されるため、テスト送信には注意してください。

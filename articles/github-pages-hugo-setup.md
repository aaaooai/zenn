---
title: "GitHub Pages + Hugo でブログを公開する"
emoji: "🐖"
type: "tech"
topics: ["Hugo", "GitHub", "Pages"]
published: true
canonical_url: https://aaaooai.github.io/posts/github-pages-hugo-setup/
---


## はじめに

おばんです。ﾀﾅｶです。会社では技術ブログを書いたりするんですが、個人では何もアウトプットできてないなぁっということに気が付きました。最近だとZennとかがいいんでしょうね、でもなんとなく、GitHub Pages+Hugoでブログ始めてみたいと思います。

## Hugo とは

Hugo は Go 言語で書かれた高速な静的サイトジェネレーターです。Markdown で記事を書くだけで、美しいブログサイトを簡単に作成できます。Hugoの公式ドキュメントに「[Host on GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/)」というのがあるので、ここを見ながら進めるとよさそうです。

## 前提条件

- Git がインストールされていること
- GitHub アカウントを持っていること
- コマンドラインの基本的な操作ができること

## 1. Hugo のインストール

ぼくは[mise](https://mise.jdx.dev/)をおすすめします。そしたらLinuxとかmacOSとかWindowsとか関係なくなるので。。。

```bash
mise use hugo@latest
```

これだけです。miseの使い方については後日別の記事でまとめましょう。すぐ試したいかたは公式ドキュメントの[Installing mise](https://mise.jdx.dev/installing-mise.html#installing-mise)をご覧ください。

## 2. 新規サイトの作成

```bash
# 新しい Hugo サイトを作成
hugo new site my-blog
cd my-blog

# Git リポジトリとして初期化
git init
```

## 3. テーマのインストール

Hugo には多くのテーマが用意されています。[ココ](https://themes.gohugo.io/)から探すか、GitHubのTopicsで「[#hugo-theme](https://github.com/topics/hugo-theme)」などを見てみるといいでしょう。

今回は軽量でシンプルな [Shibui](https://github.com/ntk148v/shibui) テーマを使用します。

```bash
# テーマをサブモジュールとして追加
git submodule add https://github.com/ntk148v/shibui.git themes/shibui

# 設定ファイルにテーマを指定
echo 'theme = "shibui"' >> hugo.toml
```

## 4. 基本設定

`hugo.toml` を編集して、サイトの基本情報を設定します。

```toml
baseURL = 'https://username.github.io/'
languageCode = 'ja-jp'
title = 'My Blog'
theme = "shibui"

[params]
  email = "your.email@example.com"

[params.author]
  name = "Your Name"

[menu]
  [[menu.main]]
    name = "Posts"
    url = "/posts/"
    weight = 1
```

## 5. 最初の記事を作成

```bash
# 新しい記事を作成
hugo new posts/first-post.md
```

`content/posts/first-post.md` を編集して、記事を書きます。

```markdown
---
title: "最初の記事"
date: 2026-02-26T21:00:00+09:00
draft: false
---

## はじめての投稿

Hugo で作成した最初の記事です。
```

`draft: false`にしているので、このままGitHubにPushしたら即座に記事が公開されます。いやいや、一回下書きに入れたいよって方はここの値を`true`にしてください。

## 6. ローカルでプレビュー

```bash
# 開発サーバーを起動（draft: true の記事も表示）
hugo server -D
```

ブラウザで `http://localhost:1313` にアクセスして、サイトを確認します。

## 7. GitHub リポジトリの作成

1. GitHub で新しいリポジトリを作成（名前は `username.github.io` にすると `https://username.github.io/` で公開できる）
2. **重要**: リポジトリは Public に設定する

## 8. GitHub Actions でデプロイ設定

`.github/workflows/hugo.yml` を作成して、GitHub Actions で自動デプロイを設定します。


```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Install mise
        uses: jdx/mise-action@v2
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 9. GitHub にプッシュ

```bash
# リモートリポジトリを追加
git remote add origin https://github.com/username/username.github.io.git

# ファイルをステージング
git add .

# コミット
git commit -m "Initial commit"

# プッシュ
git push -u origin main
```

## 10. GitHub Pages の設定

1. GitHub リポジトリの Settings > Pages に移動
2. Source を **GitHub Actions** に設定
3. しばらく待つと、GitHub Actions が自動的にデプロイを開始

## 11. サイトの確認

デプロイが完了したら、`https://username.github.io/` にアクセスして、サイトが公開されているか確認します。

## まとめ

これで Hugo を使ったブログが GitHub Pages で公開されました。

- 記事を追加する場合: `hugo new posts/new-article.md` で作成
- 更新する場合: git add, commit, push するだけで自動デプロイ

## 参考リンク

- [Hugo 公式ドキュメント](https://gohugo.io/documentation/)
- [GitHub Pages ドキュメント](https://docs.github.com/ja/pages)
- [Hugo テーマ一覧](https://themes.gohugo.io/)

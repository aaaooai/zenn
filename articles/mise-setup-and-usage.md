---
title: "mise で開発環境のツール管理をシンプルにする"
emoji: "🐖"
type: "tech"
topics: ["mise", "env", "dev", "tools"]
published: true
canonical_url: https://aaaooai.github.io/posts/mise-intro/
---


[前回](https://aaaooai.github.io/posts/github-pages-hugo-setup/)miseについて軽く話を出しました。インストール方法は次回だよっと。。。

忘れない内に書きます。

## mise とは

mise（ミーズ）は、Node.js・Python・Go・Ruby などの言語ランタイムや、各種 CLI ツールのバージョンを統一的に管理するためのツールです。よく`*env`のような名前でランタイムごとに実装されてますよね、それの代替ツールです。

mise はランタイムだけでなく、Terraform・kubectl・Helm など**バージョン管理が重要な CLI ツールも対象**です。プロジェクトごとに Terraform のバージョンを固定したいケースなどで重宝します。

もういっちょ優れている点は、`direnv` の代替としても使えるところです。`mise.toml`が置いてあるディレクトリに入るだけで自動的に読み込まれるので、`direnv` を別途インストールする必要がなくなります。

### `*env` 系との違い

`nodenv`・`pyenv`・`rbenv` など `*env` 系ツールはそれぞれ操作方法が微妙に違います。新しいツールを使うたびに使い方を調べ直す必要があります。mise は **どのツールも同じコマンドで操作できる**ので、覚えることが少なくて済みます。

### asdf との違い

asdf の置き換えを目指して開発されており、`.tool-versions` など asdf の資産はそのまま使えます。ただし、asdf からの単なる移植ではなく、**セキュリティモデルの改善**が大きな動機のひとつです。

asdf のプラグインはコミュニティの誰でも公開・管理できるため、サプライチェーンのリスクがあります。一方 mise のレジストリは作者（[@jdx](https://github.com/jdx)）のみがマージ権を持ち、デフォルトのバックエンドも asdf プラグインより [aqua](https://aquaproj.github.io/) や ubi などの検証しやすい配布方法を優先しています。詳しくは [SECURITY.md](https://github.com/jdx/mise/blob/main/SECURITY.md) を参照してください。

Rust 製なので動作が速く、`mise.toml` という設定ファイルでプロジェクトごとに使用するツールのバージョンを固定できます。

## インストール

```bash
curl https://mise.run | sh
```

インストール後、シェルの設定ファイルに以下を追加して有効化します。

```bash
# bash
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc

# zsh
echo 'eval "$(~/.local/bin/mise activate zsh)"' >> ~/.zshrc
```

設定を反映します。

```bash
source ~/.bashrc  # または ~/.zshrc
```

## 使いかた

### ツールをインストールする

```bash
# ツールの検索
mise search node

# インストールできるバージョンを確認
mise ls-remote node

# 特定バージョンを指定してインストール
mise install node@22.0.0

# 指定バージョンの最新をインストール
mise install node@22

# latest を使う
mise install node@latest

# 複数まとめてインストール
mise install node@22 python@3.12
```

### バージョンを切り替える

```bash
# グローバルに設定
mise use --global node@22

# カレントディレクトリにのみ設定（mise.toml を生成）
mise use node@22
```

### 環境変数をセットする

```bash
# kubectlにカレントディレクトリのkubeconfigを読み込ませる
mise set 'KUBECONFIG={{ config_root }}/kubeconfig'

# aws-cliのクレデンシャルを設定
mise set 'AWS_SHARED_CREDENTIALS_FILE={{ config_root }}/credentials'
```

クレデンシャルやkubeconfigなどは**絶対にリポジトリにコミットしないでください**。

シングルクォートで囲むことでシェルの展開を防ぎ、テンプレート文字列をそのまま `mise.toml` に書き込めます。mise は [Tera](https://keats.github.io/tera/) をテンプレートエンジンに採用しており、`{{ config_root }}` で `mise.toml` が置いてあるディレクトリのパスを参照できます。詳しくは mise の[テンプレート構文](https://mise.jdx.dev/templates.html)を参照してください。

```toml
[env]
KUBECONFIG = "{{ config_root }}/kubeconfig"
AWS_SHARED_CREDENTIALS_FILE = "{{ config_root }}/credentials"
```

これならリポジトリを clone した場所に関わらず正しいパスが解決されます。

### mise.toml でプロジェクトを管理する

プロジェクトルートに `mise.toml` を置くことで、チームで使用するツールのバージョンを統一できます。

```toml
[tools]
node = "22"
python = "3.12"
hugo = "latest"
```

`mise install` を実行すると `mise.toml` に書かれたツールが一括でインストールされます。

```bash
mise install
```

前回説明したGitHub Actionsの[hugo.yaml](https://aaaooai.github.io/posts/github-pages-hugo-setup/#8-github-actions-%E3%81%A7%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4%E8%A8%AD%E5%AE%9A)で`uses: jdx/mise-action@v2`の部分が`mise install`に該当してます。ここでリポジトリのmise.tomlを参照し、hugoをインストールしてます。

### 現在の設定を確認する

```bash
# インストール済みのツール一覧
mise ls

# 有効になっているツールの確認
mise current
```

このブログ自体も `mise.toml` で Hugo のバージョンを管理しています。

```toml
[tools]
hugo = "latest"
```

リポジトリを clone して `mise install` するだけで、同じ Hugo が使える状態になります。

## まとめ

- `*env` 系はツールごとに操作が違うが、mise はどれも同じコマンドで操作できる
- 言語ランタイムだけでなく Terraform・kubectl など CLI ツールのバージョン管理もできる
- `direnv` の代わりとしても使える
- asdf の資産（`.tool-versions`）はそのまま移行できる
- asdf より厳格なセキュリティモデルを採用している
- `mise.toml` をリポジトリに置くことで、チームや CI 環境でも同じバージョンを再現できる

`*env` 系や `direnv` を使ってきた人はまとめて mise に乗り換えられます。まずは `mise use` を試してみてください。

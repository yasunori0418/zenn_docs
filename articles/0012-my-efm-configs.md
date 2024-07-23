---
title: "Neovimでefmの設定管理を考える"
emoji: "🤔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Neovim
  - lua
  - lsp
  - efm
published: true
published_at: 2024-07-26
publication_name: vim_jp
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-07-26向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

今回紹介するのは、私がNeovimの中でformatterやlinterを動かすために使用しているefm-langserverというLSPの設定を管理の仕方と反省点です。
反省するということは、なにか悪いということです。（某政治家構文）
整理したいと思って、dotfilesにissueを作成しておいて、いまだに何の成果も出せていないので、問題点を書き出すついでに記事を書いてしまおうという魂胆です。

## efm-langserverの紹介と使用しているプラグイン

[efm-langserver](https://github.com/mattn/efm-langserver)はformatterやlinterをまとめてLSPのように振る舞ってくれる物です。
このLSPの設定は`$XDG_CONFIG_HOME/efm-langserver`に`config.yaml`を配置するのが通常ですが、エディタ内のLSP起動設定に含める形も可能です。
現在、私は次のプラグインを使用して、NeovimのLSP起動設定内に各種ツールの設定を含めるようにしています。

https://github.com/creativenull/efmls-configs-nvim

このプラグインはluaでefm-langserverの設定を管理できるようにするプラグインです。
デフォルトプリセット的な設定もありますし、私のようにマニュアルで設定できるように、選択的にツールの設定を制御できます。

また類似プラグインとして、次のプラグインもあります。

https://github.com/tsuyoshicho/vim-efm-langserver-settings

こちらのプラグインは内部にefm-langserver用の`config.yaml`を持っているため、こちらの方が簡単に設定できる可能性があります。

## 現在のコード

まずは執筆時点のコードを見てみましょう。

最初は実際にefm-langserverの起動設定を記述はこちら。

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/lsp/servers/efm.lua

このファイルではefm-langserverで使用したいツールの定義と、efm-langserver本体の起動設定になります。
自分のdotfilesの中ではありますが、このファイルでは定義と設定だけに整理して、
ツール設定のデータ処理は`user.plugins.efm_configs`という場所に処理をまとめています。

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/plugins/efm_configs.lua

そして、これが問題のツール設定のデータ処理系になります。
一般的なNeovimのプラグインと似たような感じに`efm_configs.setup({...})`という書き方で、
使用ツールの設定リストや、対応するファイルタイプ一覧を出力する機能を持っています。
当時はluaのクラスの書き方に慣れず、四苦八苦しながらなんとか書いた処理になります。

_「当時は…」とか言っていますが、いまだにluaでクラスを定義するのには慣れていません…。_

### 処理系の概略

処理の概略として定義したデータ型を元に、使用したいツール名や種類等を列挙して、
`setup`でツール設定を`efm-configs-nvim`から取得してLSPの起動設定に入れやすい形へ加工します。
[とある歴史的関係](https://zenn.dev/vim_jp/articles/0001-until_1st_year_engineer_can_use_my_vim)で無効にしていますが、ツールを`mason.nvim`経由でインストールすることも可能になっています。

その後、インスタンスとなった`efm_configs`から加工されたデータをプロパティとして取得する物になります。

## 後悔と反省点

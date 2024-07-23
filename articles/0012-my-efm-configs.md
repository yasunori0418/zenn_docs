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

## luaでクラスを作ってみる

### lua_ls annotations

## 反省点

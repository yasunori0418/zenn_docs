---
title: "プラグインマネージャをdpp.vimへ移行と設定方針"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Vim
  - Neovim
  - denops
  - dpp
published: false
publication_name: "vim_jp"
published_at: 2024-01-15
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-01-15向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

年末年始の休みでNeovimプラグインマネージャを[`dein.vim`][dein]から[`dpp.vim`][dpp]に移行しました。
`dein.vim`自体も難しいプラグインマネージャではありますが、`dpp.vim`はさらに上を行く難易度です。
また使用者によって構成や設定方法は違う可能性があり、応用力が求められるプラグインマネージャです。
今回の移行作業に際して、私の構成や設定方針を記事として残しておきます。

## もくもく会はいいぞ

年末年始の2023-12-28に[vim-jp][vim-jp]で開催された **『新宿もくもく会』** では、この`dpp.vim`の移行作業というテーマで参加しました。
当日以内での移行とは行きませんでしたが、もくもく会という集中できる空間もあり、よい刺激を受けながら作業ができました。
当日は悩んでいる様子しか見せることができませんでしたが、無事`dpp.vim`への移行ができたという報告も兼ねさせていただきます。

もくもく会はいいぞ！

## 構成

## 設定

### lua側の設定

### TypeScript側の設定

## 注意点

## まとめ

<!-- URLリンク集 -->

[dein]: https://github.com/Shougo/dein.vim
[dpp]: https://github.com/Shougo/dpp.vim
[vim-jp]: https://vim-jp.org

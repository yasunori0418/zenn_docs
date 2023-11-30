---
title: "ヘルプ検索から始めるddu"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Vim
  - Neovim
  - fuzzyfinder
published: true
publication_name: "vim_jp"
published_at: 2023-12-08
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2023-12-08向けの記事です。
:::
<!-- textlint-enable -->

## はじめに

皆さんVimでfuzzyfinderはお使いでしょうか？
fzf.vimでしょうか？
Neovimをお使いならtelescope.nvim？

私は闇のプラグインと名高いddu.vimをお勧めしています。

…
…
…

あー！　そこ、設定が難しいから簡単なプラグインにしたいって言った人いるでしょ！
甘いですよ。
Vimmerならプラグインの設定は避けて通れないんですから。

https://twitter.com/ShougoMatsu/status/1706991121979834792

<!-- textlint-disable -->
> 「設定させていただいてありがとうございます。」

これを言えるようにならなくては！
<!-- textlint-enable -->

設定が面倒と思ってしまう要因のひとつとして、設定項目を探すのが大変ですよね。
この難易度をグッと下げる方法として、ヘルプが検索しやすくするという方法があります。

また一説では、独りでヘルプ検索できるようになることで、初心者Vimmerを卒業している説があります。
かくいう私も最初はVimの歩き方が分からず、vim-jpに参加して操作を教わっていましたが、気がつくとひとりでもVimの操作が熟達したりプラグインの設定に躓くことが少なくなりました。

まずはdduでvimのヘルプを検索できるようになるところから設定の喜びを覚えていきましょう！

## 必要なプラグインのインストール

さて、前置きが長くなりましたが、今回必要なプラグインは次のとおりになります。

* denops.vim
  * https://github.com/vim-denops/denops.vim
  * ddu.vimを動かすためにも必須のコアプラグインです
* ddu.vim
  * https://github.com/Shougo/ddu.vim
  * これが無くてはdduは始まりません
* ddu-ui-ff
  * https://github.com/Shougo/ddu-ui-ff
  * dduをfuzzyfinderとしてのUIを出すために必要です
* ddu-filter-matcher_substring
  * https://github.com/Shougo/ddu-filter-matcher_substring
  * フィルターウィンドウで入力した文字列に応じて絞り込みを行なうために必要です
* ddu-source-help
  * https://github.com/matsui54/ddu-source-help
  * 今回の主役です。これが無いとdduでヘルプの検索はできません

`denops.vim`を動かすためにはdenoが必要です。
denoのインストール方法は、[公式ページ](https://docs.deno.com/runtime/manual/getting_started/installation)を確認してください。

またインストールに使用するプラグインマネージャは、現在お使いの物をご利用ください。
そして注意という訳ではありませんが、遅延起動などの設定は紹介しません。

まずは今回の記事を通して、dduを使ってヘルプを検索できるようになりましょう。

## 設定

次に紹介する設定を`init.vim`や`.vimrc`に追記してください。
Neovimでluaを使って設定している人もいると思いますので、luaでの設定も書いておきます。

<!-- textlint-disable -->

:::details vim scriptの場合

```vim script
call ddu#custom#patch_global(#{
\   ui: 'ff',
\   uiParams: #{
\     ff: #{
\       split: 'floating',
\       winHeight: '&lines - 8',
\       winWidth: '&columns / 2 - 2',
\       winRow: 1,
\       winCol: 1,
\       previewFloating: v:true,
\       previewHeight: '&lines - 8',
\       previewWidth: '&columns / 2 - 2',
\       previewRow: 1,
\       previewCol: '&columns / 2 + 1',
\     },
\   },
\   sourceOptions: #{
\     _: #{
\       matchers: ['matcher_substring'],
\     },
\     help: #{
\       defaultAction: 'open',
\     },
\   },
\ })

call ddu#custom#patch_local('help-ff', #{
\   sync: v:true,
\   sources: [{'name': 'help'}],
\ })

command! Help call ddu#start({'name': 'help-ff'})
```

:::
<!-- textlint-enable -->

<!-- textlint-disable -->

:::details luaの場合

:::
<!-- textlint-enable -->

## さいごに

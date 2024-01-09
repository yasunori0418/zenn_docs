---
title: "プラグインマネージャをdpp.vimへ移行と設定方針"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Vim
  - Neovim
  - lua
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

基本的な設定や使い方に関しては、[作者による記事][dpp-article]をご覧ください。

## もくもく会はいいぞ

年末年始の2023-12-28に[vim-jp][vim-jp]で開催された **『新宿もくもく会』** では、この`dpp.vim`の移行作業というテーマで参加しました。
当日以内での移行とは行きませんでしたが、もくもく会という集中できる空間もあり、よい刺激を受けながら作業ができました。
当日は悩んでいる様子しか見せることができませんでしたが、無事`dpp.vim`への移行ができたという報告も兼ねさせていただきます。

もくもく会はいいぞ！

## 構成

<!-- textlint-disable -->

::::details 過去の構成・設定について
:::message
まずは参考として、`dein.vim`を使用していたときの設定方針が[こちら](https://qiita.com/yasunori-kirin0418/items/4ac5fc07041977a8366f)にあります。
この頃から現在では設定がluaベースになったり、hooks_fileを使用するようになった影響で構成が変わっています。
過去設定との比較ということで、気になる方がいらっしゃいましたらご覧ください。
:::
::::

<!-- textlint-enable -->

### dpp.vimの拡張について

現在使用している`dpp.vim`の拡張は次のとおりです。

* [dpp-ext-lazy](https://github.com/Shougo/dpp-ext-lazy)
  * プラグインの遅延起動処理を追加する拡張
* [dpp-ext-toml](https://github.com/Shougo/dpp-ext-toml)
  * toml読み込みを使用できるようになる拡張
* [dpp-ext-installer](https://github.com/Shougo/dpp-ext-installer)
  * プラグインのインストール処理が可能になる拡張
* [dpp-protocol-git](https://github.com/Shougo/dpp-protocol-git)
  * Gitのcloneによるプラグインインストールを可能とする拡張

この拡張の構成を`dein.vim`と比較した場合、ローカルプラグインの読み込み機能を使用しない構成になります。
ローカルプラグインの読み込み機能は`dein.vim`の頃から使ってなかった機能だったので、このように使う機能を選択できるのが`dpp.vim`の良さですね。

まだ試せてはいないですが、この他にも拡張が作成されているようなので、設定のしがいがありそうですね。

### ディレクトリ構造

ディレクトリ構成は次のようにしています。

```diff : directories
 ~/.config/nvim
 ├── after
 ├── denops
+├── dpp # dpp.vimの設定をまとめたディレクトリ
+├── hooks # hooks_fileをまとめたディレクトリ
 ├── lua
+├── rc # inlineVimrcsに追加される全体設定をまとめたディレクトリ :h dpp-option-inlineVimrcs
+├── snippets # 個人で定義したスニペット用のディレクトリ(今回は関係ない…)
+└── toml # dpp-ext-tomlで読み込むtoml用のディレクトリ
```

通常の`runtimepath`や`denops.vim`を使用した場合を除いて、自動で探索されないディレクトリを追加しています。
対象のディレクトリにはマークを付けていますが、ディレクトリの名前のとおりの意味合いになっております。
このとおりに各種設定ファイルを配置し、`init.lua`や`dpp.vim`内での設定で各種ディレクトリを探索して、
すべての設定はdppのキャッシュとしてまとめられるようになっています。

設定の基本構成としては、次の流れで読み込まれていきます。

1. `init.lua`
1. `lua/user/rc.lua`
1. `dpp/config.ts`
    1. `rc/*.lua`
    1. `toml/*.toml`
    1. `hooks/*` ※各種プラグイン毎に読み込み

今回紹介する設定では、主に`lua/user/rc.lua`と`dpp/config.ts`を中心に紹介したいと思います。
tomlによる設定に関しては、`dein.vim`と構造は変わらないため説明を省きます。

## 設定

### lua側の設定

### TypeScript側の設定

## 注意点

## まとめ

<!-- URLリンク集 -->

[dein]: https://github.com/Shougo/dein.vim
[dpp]: https://github.com/Shougo/dpp.vim
[vim-jp]: https://vim-jp.org
[dpp-article]: https://zenn.dev/shougo/articles/dpp-vim-beta

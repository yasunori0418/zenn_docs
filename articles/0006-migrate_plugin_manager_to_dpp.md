---
title: "Vimプラグインマネージャー『dpp.vim』への移行と設定方針"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Vim
  - Neovim
  - lua
  - denops
  - dpp
published: true
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

基本的な設定や使い方に関しては、[Shougoさんの記事][dpp-article]をご覧ください。

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

<!-- textlint-disable -->
:::message alert
冒頭にも書きましたが、基本的な設定や使い方に関しては、[Shougoさんの記事][dpp-article]をご覧ください。
ここでは私の設定の簡単な解説までに留めます。
:::
<!-- textlint-enable -->

### lua側の設定

https://github.com/yasunori0418/dotfiles/blob/d24f946e808782290093091616fb77b81e8372fb/config/nvim/lua/user/rc.lua

`init.lua`から`lua/user/rc.lua`の`setup`関数を呼び出すことで、
各種ディレクトリのマークと必須プラグインのインストールまでをセットアップするようにしています。

#### `setup`

最初はNeovimで用意されている`NVIM_APPNAME`への対応と、各種設定を配置したディレクトリをマークするために、グローバル変数や環境変数にセットしています。
後半の方は、`dpp.vim`を使用するための設定郡で、別の関数に切り出しています。

#### `plugin_add`

一番大事な関数です。
初回起動から`dpp.vim`が使えるようになるためには、自前の極小プラグインマネージャを構築する必要があります。
私の場合色々とリッチにしていることもあって、他のGitホスティングサービスからのプラグイン追加をできるようにしていますが、
現状ではGitHubからのインストールだけで十分かと思います。

プラグインは`runtimepath`に追加されることで、プラグインとして使用可能になるため、必ず`runtimepath`への追加を行なっています。
`prepend`を使用することで`runtimepath`の先頭に`dpp.vim`として必須のプラグインが追加されるようにしています。
気持ちの問題かもしれませんが、先頭に追加するのは「そっちの方が処理は速くなるのかな…」という迷信や思いを込めています。
ここに関しては、もうすこしよい書き方があるかもしれないと模索している最中です。

#### `dpp_setup`

lua側の最後設定です。
ここでは大量に`dpp.vim`用のautocmdを定義したり、自動セットアップのために涙ぐましい努力を重ねています。
自動セットアップとして初回起動時には、すべてのプラグインがインストールされてNeovimが安定して終了するように制御しています。

`make_state`を実行するタイミングは`denops.vim`がプラグインとして使える状態になっていないといけません。
そのため、[Shougoさんの記事][dpp-article]では`DenopsReady`というイベントにフックする形で、`make_state`を実行しています。
しかし、autocmdで書かなくても`denops.vim`で用意されている`denops#server#wait_async`という関数が便利です。

```vimhelp
                                              *denops#server#wait_async()*
denops#server#wait_async({callback})
    Wait asynchronously until a |DenopsReady| autocmd is fired and invoke
    a {callback}. It invokes the {callback} immediately when the autocmd
    is already fired. If this function is called multiple times, callbacks
    registered are called in order of registration.

    DenopsReadyになるまで非同期で待機します。
    autocmdが起動され、{callback} が呼び出されます。
    autocmdがすでに起動されたら、すぐに {callback} を呼び出します。
    この関数が複数回呼び出された場合、登録されたコールバックが登録順に呼び出されます。
```

また、設定の編集・保存後にフックする形で`check_files`という処理を実行しています。
これが実行されると、`dpp.vim`で作成されるキャッシュの更新として、`make_state`を実行してくれます。
そのためNeovimの設定ファイルだけを探索しておいて、対象のファイルを変更したときだけ`check_files`を実行するようにしています。

### TypeScript側の設定

https://github.com/yasunori0418/dotfiles/blob/d24f946e808782290093091616fb77b81e8372fb/config/nvim/dpp/config.ts

`make_state`によって呼び出すTypeScriptの設定です。

基本方針として主要な処理はこの`config.ts`ファイルで実行していますが、ファイルを集めたり、型を定義するのは`helper.ts`にまとめています。
この基本方針は、`ddc.vim`と`ddu.vim`でも同じような方針で設定を分割しています。

#### ファイル探索系

各種設定ファイルを配置したディレクトリ内を探索する関数として、`helper.ts`には次の関数をそれぞれ定義しています。

* `gatherVimrcs`
* `gatherTomls`
* `gatherCheckFiles`

これらの関数では、`dpp.vim`の設定として返して欲しい型にしてデータを返すように処理をまとめて、`config.ts`では探索場所と追加の条件があれば、適宜定義しています。

これらの関数を使用するための探索場所をlua側の設定でグローバル変数や環境変数という形で定義しておくことで、`denops_std`を使えば呼び出せるということです。
たとえば`gatherVimrcs`の場合、`~/.config/nvim/rc`というディレクトリを探索しやすくするために、`vim.g.rc_dir`というグローバル変数にパスを格納しています。
これを`denops_std`で取得する場合は次のようにすると呼びだせます。

```typescript
const inlineVimrcs: string[] = gatherVimrcs(
  await vars.g.get(denops, "rc_dir"),
  vimrcSkipRules,
);
```

この`vars`というものを使えば、vim側で設定した変数へのアクセスが可能になります。

私の場合、`dpp.vim`の依存としてまとめている`deps.ts`からインポートするようにしています。

https://deno.land/x/dpp_vim/deps.ts

その後、対象のディレクトリの中のファイルリストを作成するのは、deno本体のAPIの`Deno.readDirSync`というメソッドを使用しています。

https://deno.land/api?s=Deno.readDirSync

この探索方式はtomlファイルを探索するためにも使用していますが、`gatherCheckFiles`だけは違う方法で探索しています。

`checkFiles`では設定ファイルのリストを渡す必要がありますが、設定ファイルはさまざまなディレクトリに配置されている関係で、`Deno.readDirSync`を使うのは少し大変です。
そのため設定の基礎ディレクトリとして`base_dir`というグローバル変数を定義しているので、そこから再帰的にファイルを探索する方法として、`globpath`というvimの関数を`denops_std`から呼び出しています。

```typescript
export async function gatherCheckFiles(
  denops: Denops,
  path: string,
  globs: string[],
): Promise<string[]> {
  const checkFiles: string[] = [];
  for (const glob of globs) {
    checkFiles.push(await fn.globpath(denops, path, glob, true, true));
  }

  return checkFiles.flat();
}
```

`denops_std`でvimに組み込まれた関数を使用する場合は`fn`から使用できます。
また、`globpath`という関数に関しては、`:h globpath()`でご確認ください。

#### パーシャルクローン

`dpp-protocol-git`によって、プラグインインストール時には`git clone`を使用しますが、このときに大きなプラグインを落すときに時間が掛ってしまいます。
これを短縮する方法として、「パーシャルクローン」という方式を使ってプラグインをcloneしてくれるオプションを有効にしています。
パーシャルクローンについては次の記事が参考になります。

https://github.blog/jp/2021-01-13-get-up-to-speed-with-partial-clone-and-shallow-clone/

## 移行してみての感想

`dpp.vim`移行は楽しかったのと同時に、使用できるまで思うような処理の制御が難しく、
`dein.vim`以上においそれとお勧めできないプラグインマネージャだと思いました。
それ以上にプログラマーが使うツールということを前提にしてみれば、この難易度に妥当さを感じました。

そんな高難易度プラグインマネージャですが、高難易度という言葉にワクワクしたり、設定するのが大好きっていう人は是非ともチャレンジしてみてください！

<!-- URLリンク集 -->

[dein]: https://github.com/Shougo/dein.vim
[dpp]: https://github.com/Shougo/dpp.vim
[vim-jp]: https://vim-jp.org
[dpp-article]: https://zenn.dev/shougo/articles/dpp-vim-beta

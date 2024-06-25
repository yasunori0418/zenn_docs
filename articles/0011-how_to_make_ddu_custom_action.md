---
title: "TSでdduのカスタムアクションの作り方とPRまで"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Vim
  - Neovim
  - ddu
  - fuzzyfinder
  - typescript
published: true
publication_name: vim_jp
published_at: 2024-06-28
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-06-28向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

ddu.vimをお使いのみなさん、TypeScriptで設定していますか？
ddu.vim本体や関連するプラグインは、すべてTypeScriptで実装されていますが、設定にもTypeScriptを使用できます。
今回の記事ではTypeScriptで設定することによるメリットとして、ソースに対するカスタムアクションを例として紹介していきます。
もしTypeScriptで設定していないようでしたら、少しでも参考になればよいなと思います。

### 対象読者

ddu.vimをvim scriptやluaで設定している人。

### 目標

この記事を読むことで、ddu.vimをTypeScriptによる設定が、型安全になり設定が楽しいということが分かります。

### 非目標

TypeScriptでddu.vimの設定ができるようになるところまでは紹介しません。
あくまでもTypeScriptで設定すると、型安全で設定が楽しくなるということまでが、この記事で伝えたいことです。

### 参考文献

https://zenn.dev/kamecha/articles/18d244603c85fd

過去のVim駅伝で、ddu.vimのカスタムアクションについて紹介した記事があります。
「カスタムアクションとは…？」という方は一度ご覧頂くことをお勧めします。
カスタムアクションの概要などは上記の記事をご覧ください。

一番参考になるのは[**リファレンス実装**](https://github.com/Shougo/shougo-s-github)なので、リポジトリの中から設定例を探すのが一番です。

## 前提

[参考文献](https://zenn.dev/kamecha/articles/18d244603c85fd)で紹介されている`ddu#custom#action`という関数が紹介されていますが、TypeScriptで設定する場合、`uiOptions`/`sourceOptions`/`kindOptions`で対象のUI/Source/Kindの中に`actions`というプロパティ内にアクションを定義していきます。

今回は`kindOptions`で設定していきますが、`sourceOptions`も実態としては同じで、何のsourceに対してアクションを作成したいか、という違いしかないです。

あとは`deno lsp`がエディタの中で使える状態になっていることが必須条件です。
これが使えないとメリットを享受できません。

## 設定

下記はサンプルとして、私が設定しているカスタムアクションになります。

https://github.com/yasunori0418/dotfiles/blob/6436e379/config/nvim/hooks/ddu/config.ts#L232-L260

このサンプルの中では、次のアクションを定義しています。

- uiCd
  - 選択した最初のアイテムからファイルパスを取得して、実行したいUIのアクションにパラメータとして渡す
  - [`:h ddu-source-path_history-examples`](https://github.com/Shougo/ddu-source-path_history/blob/1ad4de5/doc/ddu-source-path_history.txt#L33-L54)
- cdOpen
  - 選択した最初のアイテムからファイルパスを取得して、そのパスに`chdir`して`:edit .`を実行

`uiCd`に関してはhelpでもサンプルコードが提供されていて、内容をそのままTypeScriptで書き直しただけになります。
ですが、一番のメリットはTypeScriptによる型定義の恩恵が得られることです。

### `ActionArguments`

dduで選択したアイテムに対してアクションを実行すると、アクションには`ActionArguments`という型でデータが渡されます。
アクションは渡された`ActionArguments`のデータを元に、設定された処理を実行します。
型定義の詳細は、次のリンク先を参照してください。

https://deno.land/x/ddu_vim/types.ts?s=ActionArguments

この処理の流れはvim scriptやluaで設定していても同じことですが、TypeScriptで設定を書くことで、型定義による補完サポートの恩恵が得られます。

### `DduItem[]`と`ActionData`

前述した`ActionArguments`で一番使うものは`DduItem`内の`ActionData`です。
`ActionData`は各dduプラグインでも使用されているインターフェイスで、`ActionData`を元にKindによる処理が行なわれます。
そのため共通のデータの型ではなく、プラグイン毎に`ActionData`は定義されています。
ですが、`ActionArguments`から`ActionData`を取得しても各dduプラグインの型情報が無いため、次の用に型アサーションして使用します。

```typescript
import { ActionData as FileActionData } from "https://deno.land/x/ddu_kind_file@v0.7.1/file.ts";
// 中略
const action = args.items[0].action as FileActionData;
```

私の場合他のdduプラグインにもカスタムアクションを作成しているため、どのKindから取得してきた`ActionData`の型情報なのか分かるように、インポート時にエイリアスを設定しています。
上記の例では、[`ddu-kind-file`](https://github.com/Shougo/ddu-kind-file)の`ActionData`であることが分かるように`FileActionData`というエイリアスを設定しています。

このようにして型情報を用いながらカスタムアクションを作成できるため、デバッグログで内容を確認しながらの開発よりも、型定義に沿って開発を進めることができるため、非常に楽しく設定できます。

[***設定させていただきありがとうございます！***](https://zenn.dev/vim_jp/articles/0010-what_is_thank_you_for_allowing_setting)

## PRへ

さてここまででTypeScriptで設定することによるメリットを体感していただけたかと思いますが、これだけでは終りません。

先のカスタムアクションを定義できるということは、dduプラグインへPRを作成する手前まで来ています。
もし、貴方の作成したカスタムアクションがあれば、他の人も便利になると考えましょう。

さきほどまで作成していたカスタムアクションはそのままパッチとして、PRにすることできます。

私の場合、過去に[kyoh86/ddu-source-git_branch](https://github.com/kyoh86/ddu-source-git_branch/)へカスタムアクションを定義しつつ、他の人も使えた方が良さそうな機能としてPRを作成しました。

https://github.com/kyoh86/ddu-source-git_branch/pull/1

詳細は上記のPRのリンク先をご覧頂ければ分かるのですが、自分の設定に追加するカスタムアクションと記述の仕方は変わらないのです！

もちろん他の人にも使ってもらう関係上、ちゃんとしたクオリティに仕上げる必要はありますが、普段使っている自分の設定が便利であれば、それを共有してみたくないですか？
ddu.vimはTypeScriptで設定させていただくことができるのはもちろん、気が付けばPR作成への道標を示しているのです。

## まとめ

とりあえず、今回の記事ではddu.vimの設定をTypeScriptで記述すれば、型情報があって便利なんだなぁ～というのが伝わればOKです！

また、ddu.vimの他にもddc.vimもTypeScriptで設定が可能です。
インターフェイスは用意されていますが、まだTypeScriptで設定されている人の数が少ないため、この記事がキッカケになれば嬉しいです。

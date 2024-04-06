---
title: "ddu-gh_project作った&GitHub-Cliにコントリビュートした"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Vim
  - Neovim
  - githubcli
published: true
publication_name: "vim_jp"
published_at: 2024-04-08
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-04-08向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

とりあえず、結論としては次のとおりです。

- ddu-gh_projectでGitHub Projectをvimから操作できるようになる
- あくまでも自分が使うレベルの完成度で、他の人にも進めずらい状態
- 作る過程でGitHub-Cliにコントリビュートした（自慢）
- ghコマンドでリクエストを投げまくるので、API制限されてしまう

## ddu-gh_projectについて

普段からddu.vimを愛用しているわけですが、この度はじめてのdduプラグインを作成しました。

https://github.com/yasunori0418/ddu-gh_project

作ろうとした経緯はいろいろありますが、シンプルな理由としてはGitHub Projectをvimで操作できるようにしたかったからです。
あとは、作ってみるならghコマンドがjsonを返してくれるので簡単にできそうかもって思っていました。

結果的には簡単ではなかったです…。

### プラグイン概要

#### Source

このプラグインはddu.vimのsourceとして次の物を提供します。

1. gh_project
    - `gh project list`によって取得できるプロジェクトの一覧
1. gh_project_task
    - `gh project item-list`によって取得できる対象プロジェクトのタスク一覧

ghコマンドではプロジェクト毎のタスクは**item**という表現になっていますが、
dduのsourceも**item**という表現のため、このプラグインでは**task**という表現にしています。

#### Kind

sourceによって取得したitemに対する操作はkindになりますが、独自のkindとして次の機能を提供します。

1. gh_project
    - `openTaskList` 選択したプロジェクトのタスク一覧を開く
1. gh_project_task
    - `archive` 選択したタスクをアーカイブする
    - `create` 選択したタスクに関係無く、開いているプロジェクトにタスクを追加するためバッファーを開く
    - `deleteItem` 選択したタスクを削除
    - `edit` 選択したタスクを編集するためのバッファーを開く

現時点では、プロジェクトそのものの操作機能は提供できていません。
それ以上に優先度が高かったタスクの操作機能は実装できています。

#### タスクの作成・編集

dduのkindでは、選択したitemに対してのシングルアクションまでしかできません。
このプラグインでのユースケースでは、選択したタスクの内容を編集したいのですが、編集後の処理というステップはdduのkindでは対応範囲外です。
これを解決するために今回はdenops.vimの機能も活かしつつ、作成・編集用のtomlを生成してscratch-bufferで編集するという方法を取りました。

タスク選択からGitHub Projectへの反映までを順番に書くと次のようになります。

1. タスクの選択
    - ddu kindによって作成・編集の処理を呼び出し
1. タスクの作成・編集
    - scratch-bufferに作成・編集用のtomlをscratch-bufferに展開
1. 編集・作成内容をGitHub Projectへ反映
    - 編集内容を元にghコマンドによってクエリを実行

scratch-bufferは下書き用の一時bufferとして使用できます。

```text
:h scratch-buffer
            *scratch-buffer*
scratch     いつでも破棄されうるテキストを保持する。ウィンドウを閉じても保たれ
            明示的に削除されなければならない。
            設定は: >
            :setlocal buftype=nofile
            :setlocal bufhidden=hide
            :setlocal noswapfile
<           このバッファを識別するためにはバッファ名が使われる。
            ただし、そのためにはそのバッファに意味のある名前がついていなければならない。
```

dduによって選択したタスクを編集するためのtomlをkindで作成し、専用のscratch-bufferにインプットします。
ここで作成されるtomlには、後続のGitHub Projectへの反映処理のために必要なパラメータを含みます。

最後に編集が完了しscratch-bufferを閉じるときに、自動でscratch-bufferの内容をdenops.vim側でキャッチして、
ghコマンドによって編集内容を反映するコマンドを連続で実行していきます。

### できないこと

また、ghコマンドによって処理できないものがあります。
個人的に欲しい機能としては、`Convert to issue`という下書き状態のタスクをissueに変換する機能がGitHub Projectにはありますが、
これはghコマンドに無いため実装できていません。
工夫の仕方によってはできそうかも…とは思っていますが、まだ着手できていません。

このようにghコマンドをIFとしているため、実装したくてもできないものがあるので、必要なものがある場合はghコマンドにPRを作成する必要があります。

## GitHub-Cliへのコントリビュートについて

https://github.com/cli/cli/releases/tag/v2.46.0

![cli/cli/releases/tag/v2.46.0](https://storage.googleapis.com/zenn-user-upload/1aee57df2421-20240406.png)

今回作成したプラグインに併せて、GitHub-Cliへコントリビュートしました。
Goへのチャレンジの切っ掛けとして、とても良かったと思っています。

何よりも、色々な人が使うアプリケーションに貢献できたのと、「自慢話のネタとしてしばらく使えるなw」などと思っております。

そもそも、今回コントリビュートするまで必要だったものとは、`DraftIssueID`が`gh project item-list`に含まれていなかったからです。

### `DraftIssueID`がない

プラグインでタスクのタイトルや本文を編集したいと編集機能を作っていると、そもそもコマンドから編集できない現象に遭遇しました。

![DraftIssueID](https://storage.googleapis.com/zenn-user-upload/b9051d0082d7-20240407.png)

```text
ID must be the ID of the draft issue content which is prefixed with `DI_`
```

上では実際に、ghコマンドから取得できる`PVTI_`というプレフィックスのIDを指定してエラーになっているところです。
「もしやプレフィックスが`PVTI_`だからダメなのか？」と思い、`DI_`始まりに変えてあげたらよいのかといえば、そう甘くはなかったのです。

どうやっても編集できないため、issueを見に言ってみました。

https://github.com/cli/cli/issues/8005

まず、`DraftIssueID`とは何者かというと、GitHub Projectで作成されたタスクは`DraftIssue`という分類になります。
この`DraftIssue`をghコマンドで操作するとき、`gh project item-edit --id <item-id>`で指定する`item-id`には、`DraftIssueID`が必要だったのです。

issueでは修正方針などは決まっていて、`--format json`にしているときに`content`へ`id`という形で`DraftIssueID`が含まれるということでした。
ここまで議論が進んでいても、実装されず2023年の9月にissueはストップしていました。

> I would like the draft issue id to be included in the content that can be obtained with `gh project item-list --format json`,
> but what is the response to this issue?
> Hasn't PR etc. been created yet for this response?
> I'd like to help if possible, but I'm not familiar with Go, so it's a bit difficult.
> However, I'm currently looking at the source code to see if I can fix it somehow!
>
> `gh project item-list --format json`で取得できる内容に、ドラフト課題IDを含めて欲しいのですが、
> このissueへの対応はどうなっているのでしょうか？
> この対応についてのPRなどはまだ作成されていないのでしょうか？
> できれば協力したいのですが、Goに詳しくないのでちょっと難しいです。
> ただ、なんとか修正できないかと思い、現在ソースコードを見ているところです！

<!-- textlint-disable -->
とりあえず、質問しつつも`DraftIssueID`が無ければ始まらないため修正していきました。
確実に言えるのは、怒り駆動で修正を始めています。
まさかのGoを触る最初のプロジェクトがGitHub-CliへのPR作成になったのです。
<!-- textlint-enable -->

### PR作成からマージまで

先に書いたように修正方針は決まっていたので、生れたての小鹿の如くGoの基礎を速攻でインプットしつつ、
対象のソースを探しだして修正していきました。

修正としては、取得してきた結果の中から`DraftIssueID`を`DraftIssue`のContentに含めるだけで、残りはテストコードだけでした。

https://github.com/cli/cli/pull/8754

メンテナーのwilliammartinさんからレビューしてもらいつつ、無事PRがマージされました。
そして、このマージ後`v2.46.0`のリリースタグにて、"New Contributors"として名前を刻むことができたのでした！

## 最後に

怒り駆動でGitHub-CliへのPR作成や、想定以上に規模が大きくなったプラグインの実装は、2月の始めの頃に開始してから、3月末には落ち着いたのでした。
自分でも想像以上のスピード感で実装できたのと、自慢話のネタが増えてGoに触れる切っ掛けとなったので、
これからの目標としてはGoで何か作ってみたいと思っています。

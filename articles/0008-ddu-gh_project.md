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

### `DraftIssueID`がない

### PR作成からマージまで

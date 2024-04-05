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

- ddu-gh_projectでgithub projectをvimから操作できるようになる
- あくまでも自分が使うレベルの完成度で、他の人にも進めずらい状態
- 作る過程でgithub-cliにコントリビュートした（自慢）
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

### 苦労したこと

### こだわったところ

### できないこと

また、ghコマンドによって処理できないものがあります。
個人的に欲しい機能としては、`Convert to issue`という下書き状態のタスクをissueに変換する機能がGitHub Projectにはありますが、
これはghコマンドに無いため実装できていません。
工夫の仕方によってはできそうかも…とは思っていますが、まだ着手できていません。

このようにghコマンドをIFとしているため、実装したくてもできないものがあるので、必要なものがある場合はghコマンドにPRを作成する必要があります。

## github-cliへのコントリビュートについて

### `DraftIssueID`がない

### PR作成からマージまで

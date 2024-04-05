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

普段からddu.vimを愛用しているわけですが、この度始めてのdduプラグインを作成しました。

https://github.com/yasunori0418/ddu-gh_project

作ろうとした経緯はいろいろありますが、シンプルな理由としてはGitHub Projectをvimで操作できるようにしたかったからです。
あとは、作ってみるならghコマンドがjsonを返してくれるので簡単にできそうかもって思っていました。

### 大まかな処理の流れ

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

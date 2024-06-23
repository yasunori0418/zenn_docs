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

一番参考になるのは[**リファレンス実装**](https://github.com/Shougo/shougo-s-github)なので、リポジトリの中から設定例を探すのが一番です。

### 対象読者

ddu.vimをvim scriptやluaで設定している人。

### 目標

この記事を読むことで、ddu.vimをTypeScriptによる設定が、型安全になり設定が楽しいということが分かります。

### 非目標

TypeScriptでddu.vimの設定ができるようになるところまでは紹介しません。
あくまでもTypeScriptで設定すると、型安全で設定が楽しくなるということまでが、この記事で伝えたいことです。

## 設定

## まとめ

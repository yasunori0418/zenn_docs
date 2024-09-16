---
title: "Nixファイルの整理方法: 便利なあれこれ"
emoji: "🫖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Nix
  - NixOS
published: false
---

## 始めに

https://zenn.dev/yasunori_kirin/articles/0014-nix_file_arrangement_tips_for_import

この記事は上記の記事の続きみたいな物です。
続けて読んで貰えると嬉しいですし、単体でも読めるように書いていきます。

## let-in

Nixによって構成される設定というのは、基本的には`key-value`形式のAttrSetという物しか記述できません。
変数を宣言して活用するということを可能にするため、`let-in`という構文内で変数を宣言して、後続の構造体で利用するということが可能です。
適切に`let-in`を定義すれば、定義した変数のスコープの調整は可能になります。

https://nix.dev/manual/nix/2.18/language/constructs#let-expressions

https://zenn.dev/asa1984/books/nix-hands-on/viewer/ch01-01-nix-lang-basics#let%E5%BC%8F

文法とかの解説は上記のリンクを見てもらって、私が活用している所を紹介します。

## `++`や`//`というマージ機能の活用

## String interpolation

## まとめ

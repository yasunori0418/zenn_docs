---
title: "vimを切っ掛けにエンジニアになった話"
emoji: "🔰"
type: "idea" # tech: 技術記事 / idea: アイデア
topics:
  - ポエム
  - 初心者
  - Vim
  - Neovim
published: true
publication_name: "vim_jp"
published_at: 2023-11-27
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2023-11-27向けの記事です。
:::
<!-- textlint-enable -->

## はじめに



[前回](https://zenn.dev/vim_jp/articles/0001-until_1st_year_engineer_can_use_my_vim)エンジニアになってから1年間の成果のような記事を書きましたが、エンジニアになる切っ掛けが今回の記事になります。
というのは建前で、これはVimConf 2023 Tinyの懇親会でLTの登壇することがあり、急いで書き上げた内容を駅伝用に仕立てたものです。

## 前職は警備員

時は2020年。
当時は警備員をしていましたが、もともと子供の頃からLinuxを知る切っ掛けがあり、流れでPythonやCを簡易的に触る機会がありました。
コロナ禍ということもあり、暇潰しでエクセルVBAを使って現場の仕事の改善をやりだしたのが切っ掛けでした。
そこで収まっていたプログラミング欲が復活し、プログラミングを勉強をし始めました。

## 最初に触ったエディタはAtom

改めてプログラミングを勉強しなおそうとして使い出したエディタは、今は亡きAtomでした。
VSCodeも登場していましたがその選択は外れていました。
このときからポピュラーな物をチョイスするというのが、なぜか嫌に思うところがあったのも事実です。
しかし、gitなどのコマンドやbash操作が解説に出てくると、今度はLinux欲が復活しました。

## Linuxで生活をし続けることで、より個性的になりたくなってきた

Linuxを使い始めると、カスタマイズしたくなります。
これは自然の摂理です。
カスタマイズすることは、幸せなことなのです。
ここで出会うのはi3wmです。
つまり、キーボードだけで操作できるようになってきた訳ですね。

## Gitの勉強を続けると、ターミナルに篭りたくなる

Gitの勉強はターミナル操作がメインです。
まだAtomを使っていた私は、Atom内のターミナルと本体の動作速度に不満を覚え出し、ターミナル内の操作が増えていきました。
参考にしていたサイトの設定をそのまま使ってみると、Vimでないとコミットメッセージが書けないようになってしまいました。（まずい！）

## Vimのカスタマイズを始めてしまう（沼）

当然、初期状態のVimを使いこなせるほどの能力が無いので、カスタマイズで快適にしようと試みます。
カスタマイズ方法を知らべれば、出てくる記事はみんな`vimrc`とプラグインの導入方法です。

この中でも、「暗黒の～」や「闇のプラグイン」などと紹介されるプラグイン郡があります。
そう、Shougoさんのプラグインですね。
闇のプラグインは簡単に使えるプラグインではないですが、カスタムが素直に反映され、素早く改善される体験がとてもよいです。

## カスタマイズ沼から抜け出すどころか、常識が曲りだす

まだ警備員をやっていた私は、VimとLinuxで常識が曲ってしまいました。
快適な設定ライフを過していくとvim-jpにも出会い、警備員として生活できなくなりました。
ちょうど仕事も極端に苦しく感じるようになったので転職することを強く心に決めました。

VimとLinuxばっかりカスタマイズしていて、転職に使えるポートフォリオなんか全然ありません。
強いていうならskkeletonの入力モードをstatuslineに表示するためのプラグインだけです。
あとは今もコミットを積み続けているdotfilesだけです。

## 転職鬱を乗り越えて、エンジニア転職を成功させる

転職できたとき、私はまだ技術力が足りないと思っていました。
しかし、現実は常識が曲っていたせいで、他の未経験エンジニアよりも知識があるという「おかしい」状況になっていました。
お蔭様で、今の会社ではvimの人で通るようになりました。

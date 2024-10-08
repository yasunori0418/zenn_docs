---
title: "惰性でArchLinuxを使っていたが、必要に駆られてNixOSを使い出した"
emoji: "🫖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - nix
  - nixos
  - 入門
  - dotfiles
  - linux
published: true
---

## 始めに

エンジニア転職する少し前から、ArchLinuxをメインのOSと使い出して2年経過しました。
ArchLinuxにこれと言った不満は無く、「困ったことがあればインストールしなおしたらよいではないか」、という運用を続けていました。
実際、ネットワーク環境が整っていれば、パッケージダウンロードを含めて2時間程度で復旧できることが分かったので、惰性の運用をしていました。

ただ、最近のディストロ界隈とvim-jpの流行の波があり、世間は許してくれませんでした。
必要に駆られたため、現在はNixOSに移行して通常の作業が可能になっています。

1か月程NixOSをカスタマイズしたので、参考になりそうな情報を共有しておこうと思います。

### 前提条件

最初に書いたように、私はArchLinuxで1からデスクトップ環境を構築した経験があり、その際の知識が前提の元、現在はNixOSをメインで使用しています。
似たような人は沢山いると思っているので、是非ともNixOSに興味がある方は、お手元のArchLinuxからNixOSに移行する機運と思って参考にしてもらえると幸いです。

ArchLinuxを触ったことがある人なら、Nix言語の書き方を知ってしまえば、穴埋め問題の感覚で設定していけるはずです。

### この記事の目標

これからNixOSを始める人に向けて、参考情報を共有するまでがこの記事の役割です。
Nix言語の文法やテクニックについては、今回は共有しないこととします。

まずは、これから紹介するページやストーリーが、皆さんのNixOS開始の参考になれば幸いです。

## 参考文献

私がNixOSに移行した際、参考にしたのは次の記事です。

https://zenn.dev/asa1984/articles/nixos-is-the-best

まずは素直にこの記事どおりに、デスクトップ環境込みの状態でインストールを行ないました。
というのも、近日中にデスクトップ環境が整っていないと困る予定がありました。
そのためNixOSに慣れるという目的のためにデスクトップ環境がある状態からセットアップをしていく方針にしていきました。

いきなりArchLinuxを使うよりもManjaro辺りで慣らしてから、実際にArchLinux Install Battleするのと似ていますね。

あとは基本を知るという意味で、Nixその物の入門記事として次の記事を暇があれば読んでいます。

https://zenn.dev/asa1984/books/nix-introduction

主にNix言語について知らないことだらけのため、足りない言語知識を埋めるため、言語についての解説ページは何度も見直していました。

あと、Nixを私に勧めてくれた[たけてぃ](https://github.com/takeokunn)のNixOSの設定も参考にさせてもらいました。

https://github.com/takeokunn/nixos-configuration

あとはパッケージ検索兼、設定項目の検索もできる公式の検索サイトは手放せないでしょう。

https://search.nixos.org/packages

また、NixOS自体の設定項目の一覧は、Web上で確認できる他、`man`コマンドでも確認できます。

https://nixos.org/manual/nixos/stable/options

```bash
man 5 configuration.nix
```

## 設定途中の躓き

基本はAsahiさんのデスクトップ環境を構築する記事をベースに設定していき、徐々に普段使っているi3wm環境に置き換えていく作業を行なっていきました。
私のdotfilesではシェルスクリプトやCLIツールをダウンロードして環境を構築するようにしていたのですが、NixOSという環境では簡単に使わせてくれませんでした。

### nix-ld

私のdotfilesでは次のようなツール郡を使用して、開発環境を整えるようにしていました。

- aqua（cliツール管理）
  - shell関連のツールはaquaからインストール
- mise（言語ランタイム管理）
- Neovim

これらのツールでインストールした物を利用し、メインとなるテキストエディタの環境を補助していた訳ですが、NixOSの環境では外部から落してきたバイナリは動いてくれません。
NixOSはFHSに準拠していないため、直接実行可能なバイナリを使用すると共有ライブラリが見つからないため、動作しないという現象が発生してしまいます。

```
❯ nvim
Could not start dynamically linked executable: nvim
NixOS cannot run dynamically linked executables intended for generic
linux environments out of the box. For more information, see:
https://nix.dev/permalink/stub-ld

---

動的にリンクされた実行可能ファイルを開始できませんでした: nvim
NixOS は、汎用向けに動的にリンクされた実行可能ファイルを実行できません
すぐに使える Linux 環境。 詳細については、以下を参照してください。
https://nix.dev/permalink/stub-ld
```

上記の内容は、実際にNixOS上でreleaseページからダウンロードしてきたNeovimを使用したときに出るメッセージです。
エラー内容のリンク先を見に行くと解決方法を教えてくれます。

https://nix.dev/permalink/stub-ld

ざっくり言ってしまえば、次の設定を`/etc/nixos/configuration.nix`に追記して、`nixos-rebuild switch`の実行と、再起動をしてみてください。

```nix
programs.nix-ld.enable = true;
```

何ごともなく落してきたバイナリが動きます！
私が使用している範囲では、上記の設定を入れるだけで問題なく動作しています。

類似内容だと、こちらも参考にしてみてください。

https://zenn.dev/asa1984/scraps/17fe60c1b2ccc2

## 現在の設定状況

OSのカスタムとしては1～2週間程あれば、概ね満足のできる環境が構築できるはずです。
とりあえず、動く状態の設定に変更できれば、あとは設定を分割して整理する作業を繰り返しているはずです。
この記事を書いている時点の私の設定を共有しておきます。

https://github.com/yasunori0418/dotfiles/tree/b629c960/nixos

OSとしての設定は`nixos`というディレクトリにまとめていて、各環境の設定はマシン名のディレクトリを作成し、マシン固有の設定は対象のディレクトリ内で定義しています。
それ以外の共通で使える設定は`modules`というディレクトリに配置しています。

`modules`内のディレクトリ構成も確立してきている状態です。
設定の読み込みの仕方や、分割方法は現状の状態で確定だと思っています。

よかったら参考にしてみてください。

## まとめと、これからNixOSを始める人へ

いろいろ書きましたが、まずは参考記事から読んで、NixOSの環境を作ってみてください。

ここ1か月触ってみての感想は、Nix言語によってOSの設定をさせていただけるという感覚です。
何よりも大事なのはNix言語を覚えて、いろいろ試してみることだと思います。

あと一番の助けになったのは、周辺にNixを触っている人がいて、すぐに相談できる環境が大事です。
日本語公式コミュニティがあり、Discordのサーバーがあるので入ってみるとよいと思います。
nixpkgsのコミッターをしている方がいるので、非常に参考になります。

https://github.com/nix-ja

私の場合は、vim-jpで流行りだしたのが切っ掛けなので、[vim-jp slack](https://vim-jp.org/docs/chat.html)の`#tech-nix`チャンネルに入ってみるのもお勧めします。
熱狂的に設定するのが大好きな人達のお陰で、毎日情報共有されています。

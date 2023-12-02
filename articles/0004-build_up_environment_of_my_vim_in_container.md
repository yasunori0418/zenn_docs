---
title: "最高のvim環境を構築するためのdocker術"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Vim
  - Neovim
  - lua
  - docker
published: true
publication_name: "vim_jp"
published_at: 2023-12-03
---

## はじめに

ここ最近、仕事で「自分のVim」を使えるようになりました。
これを使えるようにする方法として、dockerコンテナの中に自分のVimを動作させる環境を構築して開発時に使用しています。
私と似たような制限された状況になった場合などの回避策として、参考になったらよいなと思って記事にしておきます。

## 自分のVimを知る

コンテナの中にVimをインストールすると言っても、普段から使っているVimを把握していなくてはコンテナ内に環境構築などできません。
まずは自分が普段どのようなVimを使っているか確認しましょう。
確認観点としては次のようなパターンが考えられます。

1. Vim or Neovim
    * `/\c\(neo\|n\)\?vim`
    * 基礎中の基礎ではありますが、大事なところです
1. 使用しているVimのバージョン
    * Stable
    * nightly
    * HEAD
1. 設定のディストロを使っているか
    * SpaceVim, LunarVim and etc...
    * **1から設定をフルスクラッチしている**
        * *ディストロなど甘え*
1. 依存している言語・ツール
    * Python等の外部言語を使用しているか
    * fzfやfd,rgなどを使用するように設定しているか

この確認観点の中でも依存している言語やツールというのは、特別整理してリストアップしておくことをお勧めします。
また設定のディストロがありますが、SpaceVimやLunarVimといった設定のディストロを使っている方は、そこで依存関係を確認してください。
設定のディストロを使っている方はメンテナンスが行き届いていれば、依存関係の整理はさほど難しくはないでしょう。
次の章での解説になりますが、プラグインマネージャの初回セットアップからプラグインのインストールまでをお膳立てしてくれているようなものだと思いますので、設定のディストロのセットアップ手順を確認しておいてください。
私は設定のディストロを使ったことが無いので、あまり助けにはなりませんが…。

ここで一番大変かつ、環境構築の醍醐味といえるのは「1から設定を書いたオレオレディストロ」をお使いの方ですね。
まぁ、逆に「1から設定を書く人」であれば、大変さを気にする必要は無いかもしれませんね（笑）

## プラグインマネージャのインストールからセットアップまでを自動化しておく

プラグインのインストールはプラグインマネージャに任せていることでしょう。
「そのプラグインマネージャのインストールは？」となるので、ここを自動化しておきましょう。
極稀にプラグインマネージャをdotfilesにまとめてコミットされている方もいますが、GitHubなどでコード検索したときに混乱の元になるので避けておきたいですね。

次のコードはNeovimでluaを使って、プラグインマネージャのインストールまでを自動化する関数です。

```lua
---初回起動時にプラグインのダウンロードとruntimepathに追加する
---@param host string e.g. github.com
---@param repo string user_name/plugin_name
local function init_plugin(host, repo)
    -- M.dein_dir = vim.fs.joinpath(vim.env.XDG_CACHE_HOME, "dein")
    local repo_dir = M.dein_dir .. "/repos/" .. host .. "/" .. repo
    local plugin_name = vim.fn.split(repo, "/")[2]
    if not vim.regex("/" .. plugin_name):match_str(vim.o.runtimepath) then
        if vim.fn.isdirectory(repo_dir) ~= 1 then
            os.execute("git clone https://" .. host .. "/" .. repo .. " " .. repo_dir)
        end
        vim.opt.runtimepath:append(repo_dir)
    end
end
```

私は`dein.vim`を使っているので、そのディレクトリ構造を想定した場所にプラグインをインストールするようにしています。
私はこの関数を`init.vim`内で呼び出し、`dein.vim`の他に`vim-artemis`を初回起動時にインストールしています。
つまり、この関数はミニプラグインマネージャになるのです。
当然ながらインストールは初回のみで、2回目以降の起動時にはプラグインのディレクトリの存在を確認できるので、`runtimepath`にプラグインが追加されるだけになります。

また、各プラグインマネージャによって想定するディレクトリ構造は違うので、プラグインマネージャの仕様を確認しておくのがよいでしょう。
必要最低限のプラグインのインストール以降は、プラグインマネージャのインストール機能を活用して、残りのプラグイン達をインストールしてもらいましょう。

## 使いやすいディストロを選択する

通常dockerコンテナ内の環境というのは、最小構成にするのが流儀ではあります。
そのためにコンテナ用のイメージは、パッケージ数が極端に少ない場合があります。
今回作るコンテナ内にて自分のVimで開発するのが目標のため、コンテナ内を最小構成にするという考えを捨てた方がよいと思います。

ポピュラーなディストロを選択するなら、最初の候補に挙がるのはUbuntuでしょう。
aptの情報も多いため、構築が簡単かと思います。
ですが、Vimをコンテナ内でビルドしたい場合や、特定の言語での開発するときに結果的に余計な肥大化をしてしまったため、私は最終的に`archlinux:base-devel`を使っています。

コアなVimmerは比較的ArchLinuxを使っている比率が高いので、「1から設定を書いている人」であれば抵抗は少ないと思われます。
ArchLinuxでもAURというリポジトリからのインストールになることは少なく、base-develであればビルドツールがひととおり揃っているので、コンテナ内で最新のVimを使って開発するということが可能になります。

Neovimをコンテナ内でビルドしたかったので私は次のようなパッケージリストを作成して、Dockerfile内でインストールするようにしています。

```text
cmake
curl
git
ninja
unzip
vim
wget
xdg-user-dirs
zsh
```

```Dockerfile
FROM archlinux:base-devel

# Load package list file with variable.
ARG PKGLIST_FILE=./.docker/pkglist.txt

# You can specify a list of non-aur packages in text format.
# Adding package list.
COPY ${PKGLIST_FILE} /etc/pacman.d/${PKGLIST_FILE}

ARG COUNTRY=ja_JP
ARG ENCODE=UTF-8
ENV TZ=Asia/Tokyo
ENV SHELL=/usr/bin/zsh
WORKDIR /root

# The pacman-key and pacman -Syu commands must be run for the initial time with archlinux images.
RUN pacman-key --init && \
    pacman-key --populate archlinux && \
    pacman -Syu --noconfirm && \
    pacman -S --needed --noconfirm - < /etc/pacman.d/${PKGLIST_FILE} && \
    xdg-user-dirs-update && \
    ln -svf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo "LANG=${COUNTRY}.${ENCODE}" > /etc/locale.conf
```

とりあえず参考として、私が使っているArchLinuxのDockerfileを紹介してみましたが、皆さんがお使いのOSに合わせるのが最終的にはよいと思います。
是非、「自分のVim」が動作する素敵なコンテナを作成してみてください。

## 私は"dotfiles in docker: 通称did"を作った

ここまで紹介したのは、あくまでもVimを動かすだけを目標にした内容です。
私の場合、dotfilesを育てるのも大好きなので、dotfilesに詰めた開発のための機能をすべて動作させる目的でコンテナを構築しています。

https://github.com/yasunori0418/did

現状だれかに使って貰える想定にはしていませんが、`Makefile`でよく実行する操作はまとめています。

```terminal
$ make
help                          subcommand list and description.
start                         docker compose start
stop                          docker compose stop
ps                            docker compose ps
zsh                           container start and attach with zsh
init                          Initialize my favorite environment container.
clean                         Remove all container information.
reset                         clean & init
```

これをVimConf2023の前日のオフ会で紹介したところ、「だれでも穿けるパンツ」という称号を得ました。

### What is `pants`?

なぜこれがパンツと呼ばれるのか、それには理由があります。
[vim-jp](https://vim-jp.org/)ではしばしば、他人の設定やdotfilesをパンツと比喩することがあります。
つまり、私が作った`did`というのは「だれでも穿けるパンツ」ということになるのです。

この人のパンツという比喩の生まれを把握している訳ではないですが、恐らくは「人の褌で相撲を取る」という慣用句が由来としているのかと思われます。
こちらの慣用句をちゃんと調べてみました。

https://kotobank.jp/word/%E4%BA%BA%E3%81%AE%E8%A4%8C%E3%81%A7%E7%9B%B8%E6%92%B2%E3%82%92%E5%8F%96%E3%82%8B-611187

> 他人の物を利用して、自分の事に役立てる。

という意味になります。
ほかにも次のような慣用句があるようです。

> 人の太刀で功名する。
> 人の提灯で明りを取る。

どうでしょうか、「他人の物を利用して…」などといわれるよりも、自分で設定を書いた方がよいと思いませんか？

## さいごに

必要性を感じて依存関係を増やしていれば納得できる構成になりますが、設定のディストロを使っていると訳も分からず依存関係を増やすことになるので、個人的な意見を言ってしまえば管理できていないと思ってしまいます。
これを機に自分で設定を書いてみませんか？

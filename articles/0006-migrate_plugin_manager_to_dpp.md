---
title: "プラグインマネージャをdpp.vimへ移行と設定方針"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Vim
  - Neovim
  - lua
  - denops
  - dpp
published: false
publication_name: "vim_jp"
published_at: 2024-01-15
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-01-15向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

年末年始の休みでNeovimプラグインマネージャを[`dein.vim`][dein]から[`dpp.vim`][dpp]に移行しました。
`dein.vim`自体も難しいプラグインマネージャではありますが、`dpp.vim`はさらに上を行く難易度です。
また使用者によって構成や設定方法は違う可能性があり、応用力が求められるプラグインマネージャです。
今回の移行作業に際して、私の構成や設定方針を記事として残しておきます。

基本的な設定や使い方に関しては、[作者による記事][dpp-article]をご覧ください。

## もくもく会はいいぞ

年末年始の2023-12-28に[vim-jp][vim-jp]で開催された **『新宿もくもく会』** では、この`dpp.vim`の移行作業というテーマで参加しました。
当日以内での移行とは行きませんでしたが、もくもく会という集中できる空間もあり、よい刺激を受けながら作業ができました。
当日は悩んでいる様子しか見せることができませんでしたが、無事`dpp.vim`への移行ができたという報告も兼ねさせていただきます。

もくもく会はいいぞ！

## 構成

<!-- textlint-disable -->

::::details 過去の構成・設定について
:::message
まずは参考として、`dein.vim`を使用していたときの設定方針が[こちら](https://qiita.com/yasunori-kirin0418/items/4ac5fc07041977a8366f)にあります。
この頃から現在では設定がluaベースになったり、hooks_fileを使用するようになった影響で構成が変わっています。
過去設定との比較ということで、気になる方がいらっしゃいましたらご覧ください。
:::
::::

<!-- textlint-enable -->

### dpp.vimの拡張について

現在使用している`dpp.vim`の拡張は次のとおりです。

* [dpp-ext-lazy](https://github.com/Shougo/dpp-ext-lazy)
  * プラグインの遅延起動処理を追加する拡張
* [dpp-ext-toml](https://github.com/Shougo/dpp-ext-toml)
  * toml読み込みを使用できるようになる拡張
* [dpp-ext-installer](https://github.com/Shougo/dpp-ext-installer)
  * プラグインのインストール処理が可能になる拡張
* [dpp-protocol-git](https://github.com/Shougo/dpp-protocol-git)
  * Gitのcloneによるプラグインインストールを可能とする拡張

この拡張の構成を`dein.vim`と比較した場合、ローカルプラグインの読み込み機能を使用しない構成になります。
ローカルプラグインの読み込み機能は`dein.vim`の頃から使ってなかった機能だったので、このように使う機能を選択できるのが`dpp.vim`の良さですね。

まだ試せてはいないですが、この他にも拡張が作成されているようなので、設定のしがいがありそうですね。

### ディレクトリ構造

ディレクトリ構成は次のようにしています。

```diff : directories
 ~/.config/nvim
 ├── after
 ├── denops
+├── dpp # dpp.vimの設定をまとめたディレクトリ
+├── hooks # hooks_fileをまとめたディレクトリ
 ├── lua
+├── rc # inlineVimrcsに追加される全体設定をまとめたディレクトリ :h dpp-option-inlineVimrcs
+├── snippets # 個人で定義したスニペット用のディレクトリ(今回は関係ない…)
+└── toml # dpp-ext-tomlで読み込むtoml用のディレクトリ
```

通常の`runtimepath`や`denops.vim`を使用した場合を除いて、自動で探索されないディレクトリを追加しています。
対象のディレクトリにはマークを付けていますが、ディレクトリの名前のとおりの意味合いになっております。
このとおりに各種設定ファイルを配置し、`init.lua`や`dpp.vim`内での設定で各種ディレクトリを探索して、
すべての設定はdppのキャッシュとしてまとめられるようになっています。

設定の基本構成としては、次の流れで読み込まれていきます。

1. `init.lua`
1. `lua/user/rc.lua`
1. `dpp/config.ts`
    1. `rc/*.lua`
    1. `toml/*.toml`
    1. `hooks/*` ※各種プラグイン毎に読み込み

今回紹介する設定では、主に`lua/user/rc.lua`と`dpp/config.ts`を中心に紹介したいと思います。
tomlによる設定に関しては、`dein.vim`と構造は変わらないため説明を省きます。

## 設定

<!-- textlint-disable -->
:::message alert
冒頭にも書きましたが、基本的な設定や使い方に関しては、[作者による記事][dpp-article]をご覧ください。
ここでは私の設定の簡単な解説までに留めます。
:::
<!-- textlint-enable -->

### lua側の設定

`init.lua`から`lua/user/rc.lua`を呼び出すことで、各種ディレクトリのマークと必須プラグインのインストールまでをセットアップするようにしています。

<!-- textlint-disable -->

:::details setup

```lua :lua/user/rc.lua
---init.luaで呼び出すdpp.vimの初期設定
---NVIM_APPNAMEを使ってプロファイルとして分離してみる
---NVIM_APPNAMEが設定されていない場合は、デフォルトの`nvim`になる
function M.setup()
    M.nvim_appname = vim.env.NVIM_APPNAME or "nvim"
    if M.nvim_appname == "nvim" then
        M.dpp_dir = joinpath(vim.env.XDG_CACHE_HOME, "dpp")
    else
        M.dpp_dir = joinpath(vim.env.XDG_CACHE_HOME, M.nvim_appname .. "_dpp")
    end

    vim.g.base_dir = joinpath(vim.env.XDG_CONFIG_HOME, M.nvim_appname)
    vim.env.BASE_DIR = vim.g.base_dir

    vim.g.hooks_dir = joinpath(vim.g.base_dir, "hooks")
    vim.env.HOOKS_DIR = vim.g.hooks_dir
    vim.g.snippet_dir = joinpath(vim.g.base_dir, "snippets")
    vim.g.rc_dir = joinpath(vim.g.base_dir, "rc")
    vim.g.toml_dir = joinpath(vim.g.base_dir, "toml")

    plugin_add("Shougo/dpp-ext-lazy")
    plugin_add("Shougo/dpp-ext-toml")
    plugin_add("Shougo/dpp-ext-installer")
    plugin_add("Shougo/dpp-protocol-git")
    plugin_add("Shougo/dpp.vim")
    plugin_add("vim-denops/denops.vim")
    dpp_setup()
end
```

:::

<!-- textlint-enable -->

主に前半部分は、Neovimで用意されている`NVIM_APPNAME`への対応と、
各種設定を配置したディレクトリをマークするために、グローバル変数や環境変数にセットしています。

一番大事なのは`plugin_add`関数です。
初回起動から`dpp.vim`が使えるようになるためには、自前の極小プラグインマネージャを構築する必要があります。

<!-- textlint-disable -->

:::details plugin_add

```lua : lua/user/rc.lua
---@diagnostic disable: duplicate-doc-alias
---@alias plugin_add_type
---| "prepend"
---| "append"

---初回起動時にプラグインのダウンロードとruntimepathに追加する
---@param repo string user_name/plugin_name
---@param host? string default: github.com
---@param type? plugin_add_type user_name/plugin_name
local function plugin_add(repo, host, type)
    host = host or "github.com"
    type = type or "prepend"
    local repo_dir = joinpath(M.dpp_dir, "repos", host, repo)
    local plugin_name = vim.fn.split(repo, "/")[2]
    if not vim.regex("/" .. plugin_name):match_str(vim.o.runtimepath) then
        if vim.fn.isdirectory(repo_dir) ~= 1 then
            os.execute("git clone https://" .. host .. "/" .. repo .. " " .. repo_dir)
        end
        if type == "prepend" then
            vim.opt.runtimepath:prepend(repo_dir)
        else
            vim.opt.runtimepath:append(repo_dir)
        end
    end
end
```

:::

<!-- textlint-enable -->

私の場合色々とリッチにしていることもあって、他のGitホスティングサービスからのプラグイン追加をできるようにしていますが、
現状ではGitHubからのインストールだけで十分かと思います。

プラグインは`runtimepath`に追加されることで、プラグインとして使用可能になるため、必ず`runtimepath`への追加を行なっています。
`prepend`を使用することで`runtimepath`の先頭に`dpp.vim`として必須のプラグインが追加されるようにしています。
気持ちの問題かもしれませんが、先頭に追加するのは「そっちの方が処理は速くなるのかな…」という迷信や思いを込めています。

lua側の設定最後は、`dpp_setup`です。
ここでは大量にautocmdを定義したり、自動セットアップのために涙ぐましい努力を重ねています。

<!-- textlint-disable -->

:::details dpp設定関連

```lua : lua/user/rc.lua
local function gather_check_files()
    local glob_patterns = {
        "**/*.lua",
        "**/*.toml",
        "**/*.ts",
    }
    local check_files = {}
    for _, glob_pattern in pairs(glob_patterns) do
        table.insert(check_files, vim.fn.globpath(vim.g.base_dir, glob_pattern, true, true))
        table.insert(check_files, vim.fn.globpath("~/dotfiles/config/nvim", glob_pattern, true, true))
    end
    return vim.tbl_flatten(check_files)
end

local function auto_install_plugins(dpp)
    local notInstallPlugins = vim.iter(vim.tbl_values(dpp.get()))
        :filter(function(p)
            return vim.fn.isdirectory(p.rtp) == 0
        end)
        :totable()
    if #notInstallPlugins > 0 then
        vim.fn["denops#server#wait_async"](function()
            dpp.async_ext_action("installer", "install")
        end)
    end
end

local function dpp_setup()
    local dpp = require("dpp")
    local rc_autocmds = vim.api.nvim_create_augroup("RcAutocmds", { clear = true })
    if dpp.load_state(M.dpp_dir) > 0 then
        vim.fn["denops#server#wait_async"](function()
            dpp.make_state(M.dpp_dir, joinpath(vim.g.base_dir, "dpp", "config.ts"), M.nvim_appname)
            vim.api.nvim_create_autocmd("User", {
                pattern = "Dpp:makeStatePost",
                group = rc_autocmds,
                callback = function()
                    dpp.load_state(M.dpp_dir)
                    auto_install_plugins(dpp)
                    vim.api.nvim_create_autocmd("User", {
                        pattern = "Dpp:makeStatePost",
                        group = rc_autocmds,
                        callback = function()
                            vim.cmd.quit({ bang = true })
                        end,
                    })
                end,
                once = true,
                nested = true,
            })
        end)
    else
        vim.api.nvim_create_autocmd("BufWritePost", {
            pattern = gather_check_files(),
            group = rc_autocmds,
            callback = function()
                vim.notify("dpp check_files() is run", vim.log.levels.INFO)
                dpp.check_files()
            end,
        })
    end
    auto_install_plugins(dpp)
    vim.api.nvim_create_autocmd("User", {
        pattern = "Dpp:makeStatePost",
        group = rc_autocmds,
        callback = function()
            vim.notify("dpp make_state() is done", vim.log.levels.INFO)
        end,
    })
end
```

:::

<!-- textlint-enable -->

### TypeScript側の設定

## 注意点

## まとめ

<!-- URLリンク集 -->

[dein]: https://github.com/Shougo/dein.vim
[dpp]: https://github.com/Shougo/dpp.vim
[vim-jp]: https://vim-jp.org
[dpp-article]: https://zenn.dev/shougo/articles/dpp-vim-beta

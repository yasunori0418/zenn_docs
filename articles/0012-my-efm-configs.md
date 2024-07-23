---
title: "Neovimでefmの設定管理を考える"
emoji: "🤔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Neovim
  - lua
  - lsp
  - efm
published: true
published_at: 2024-07-26
publication_name: vim_jp
---

<!-- textlint-disable -->
:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)の2024-07-26向けの記事です。
:::
<!-- textlint-enable -->

## 始めに

今回紹介するのは、私がNeovimの中でformatterやlinterを動かすために使用しているefm-langserverというLSPの設定を管理の仕方と反省点です。
反省するということは、なにか悪いということです。（某政治家構文）
整理したいと思って、dotfilesにissueを作成しておいて、いまだに何の成果も出せていないので、問題点を書き出すついでに記事を書いてしまおうという魂胆です。

## efm-langserverの紹介と使用しているプラグイン

[efm-langserver](https://github.com/mattn/efm-langserver)はformatterやlinterをまとめてLSPのように振る舞ってくれる物です。
このLSPの設定は`$XDG_CONFIG_HOME/efm-langserver`に`config.yaml`を配置するのが通常ですが、エディタ内のLSP起動設定に含める形も可能です。
現在、私は次のプラグインを使用して、NeovimのLSP起動設定内に各種ツールの設定を含めるようにしています。

https://github.com/creativenull/efmls-configs-nvim

このプラグインはluaでefm-langserverの設定を管理できるようにするプラグインです。
デフォルトプリセット的な設定もありますし、私のようにマニュアルで設定できるように、選択的にツールの設定を制御できます。

また類似プラグインとして、次のプラグインもあります。

https://github.com/tsuyoshicho/vim-efm-langserver-settings

こちらのプラグインは内部にefm-langserver用の`config.yaml`を持っているため、こちらの方が簡単に設定できる可能性があります。

## 現在のコード

まずは執筆時点のコードを見てみましょう。

最初は実際にefm-langserverの起動設定を記述はこちら。

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/lsp/servers/efm.lua

このファイルではefm-langserverで使用したいツールの定義と、efm-langserver本体の起動設定になります。
自分のdotfilesの中ではありますが、このファイルでは定義と設定だけに整理して、
ツール設定のデータ処理は`user.plugins.efm_configs`という場所に処理をまとめています。

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/plugins/efm_configs.lua

そして、これが問題のツール設定のデータ処理系になります。
一般的なNeovimのプラグインと似たような感じに`efm_configs.setup({...})`という書き方で、
使用ツールの設定リストや、対応するファイルタイプ一覧を出力する機能を持っています。
当時はluaのクラスの書き方に慣れず、四苦八苦しながらなんとか書いた処理になります。

*「当時は…」とか言っていますが、いまだにluaでクラスを定義するのには慣れていません…。*

### 処理系の概略

処理の概略として定義したデータ型を元に、使用したいツール名や種類等を列挙して、
`setup`でツール設定を`efm-configs-nvim`から取得してLSPの起動設定に入れやすい形へ加工します。
[とある歴史的関係](https://zenn.dev/vim_jp/articles/0001-until_1st_year_engineer_can_use_my_vim)で無効にしていますが、ツールを`mason.nvim`経由でインストールすることも可能になっています。

その後、インスタンスとなった`efm_configs`から加工されたデータをプロパティとして取得する物になります。

## 後悔と反省点

この段階では特に問題という問題が無いように思いますが、こちらをご覧ください。

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/lsp/servers/efm.lua#L21-L28

https://github.com/yasunori0418/dotfiles/blob/8b67843/config/nvim/lua/user/lsp/servers/efm.lua#L40-L58

***おわかりいただけただろうか…***

類似するファイルタイプで、使用したいツールが同じパターンなのです！

これは純粋に考慮漏れから来る設計ミスな訳ですが、見方を変えたら冗長的に設定を宣言できている…？
とは言っても一度プラグインから取得してきたテーブルを再度呼びだすのはパフォーマンス的には問題だと思ていたり…。

**そもそも処理を分けた理由は、同じような記述を連続して列挙したくなかったのと、プラグインとして切り出せる可能性を作っておきたかったからです。**

使用ツールに対して、複数のファイルタイプを設定できるようなデータ構造にしないといけない。
部署移動した関係でMacを遊びだして改善に手を出せず幾星霜…。

で、反省点を書くという記事を書いているって訳。

## 改善案

luaのannotationには慣れていないけど、TypeScriptなら雰囲気で書けるので、多分こんな感じの型を`setup`が受け入れるようにしようと考えている物はあります。

```typescript
interface ToolConfig {
  toolKind: "formatters" | "linters";
  name: string;
  filetypes: string[];
  autoInstall?: boolean;
}

interface EfmConfigSetup {
  autoInstalls: boolean;
  tools: ToolConfig[];
}
```

一番の肝は`ToolConfig`ですね。
🤖「最初から使いたいツール情報をまとめた配列にしておけば良かったのに…。」

ただ、レコードにしておくことでファイルタイプが、キーの一覧を取得したらLSPの設定に入れやすいというメリットがあります。
使用している[`efm-configs-nvim`のREADME](https://github.com/creativenull/efmls-configs-nvim#setup)でも同じような構成で設定が紹介されています。

```lua
-- Register linters and formatters per language
local eslint = require('efmls-configs.linters.eslint')
local prettier = require('efmls-configs.formatters.prettier')
local stylua = require('efmls-configs.formatters.stylua')
local languages = {
  typescript = { eslint, prettier },
  lua = { stylua },
}

-- Or use the defaults provided by this plugin
-- check doc/SUPPORTED_LIST.md for the supported languages
--
-- local languages = require('efmls-configs.defaults').languages()

local efmls_config = {
  filetypes = vim.tbl_keys(languages),
  settings = {
    rootMarkers = { '.git/' },
    languages = languages,
  },
  init_options = {
    documentFormatting = true,
    documentRangeFormatting = true,
  },
}

require('lspconfig').efm.setup(vim.tbl_extend('force', efmls_config, {
  -- Pass your custom lsp config below like on_attach and capabilities
  --
  -- on_attach = on_attach,
  -- capabilities = capabilities,
}))
```

## 最後に

時間を作って改善案のデータ型で処理する形に変更したいと考えています。
幸いにも記事を書いて、反省点や改善案を出せたので、次のVim活のネタが捻り出せたのは良かったです！

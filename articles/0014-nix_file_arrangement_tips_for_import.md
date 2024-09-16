---
title: "Nixファイルの整理方法: imports/import関数"
emoji: "🫖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - Nix
  - NixOS
published: true
---

## 始めに

[前回書いたNixOSの記事](https://zenn.dev/yasunori_kirin/articles/0013-nixos-first-setup)の評判が良く、
「少しでもNixへチャレンジしてくれる人が増えてくれたら嬉しいな」と思う今日この頃です。

さて、今回は個人利用の範囲では常用運用できているNixOSですが、
どのように設定ファイルを分割して保守しているのか備忘録的にまとめたいと思います。

### 前提

https://zenn.dev/asa1984/books/nix-hands-on

早速ですが、先日公開された「Nix入門：ハンズオン」は是非一読していただきたいです。
上記の本で紹介されているNix言語の文法を元に、私が使用している内容に触れていきます。
~~この記事ではNixOSやhome-managerといった物に触れるということはせず、ファイル分割や設定のまとめ方を共有していきます。~~

…で、書いている最中に文章量がえげつないことになってきたので、今回は`import関数`と`imports`に絞っていきます。

Nix言語に触れていると最初に遭遇する紛らわしい物です。
この2つの違いと活用方法を見ていきましょう。

<!-- textlint-disable -->
:::message
**追記 2024/09/16 16:00**

この記事における`imports`の説明は主にNixOSのモジュールシステムとしての物になります。
詳細については、「[補足（home-managerについて）](#補足（home-managerについて）)」の章をご覧ください。
※ 以上、[natsukium氏](https://github.com/natsukium)からの~~天の声~~情報提供でした。
:::
<!-- textlint-enable -->

### 対象読者

<!-- textlint-disable -->
- 言われたまま設定を書いているけど、そろそろ整理したいと考えている人
- ファイルは分割しないと落ち着かない人
- Nix言語に慣れていきたい人

下記は長すぎるため、更に折り畳みに折り畳んでいます。
::::details この状態が我慢できない人

これは私の`/etc/nixos/configuration.nix`の残骸です。
現在は、flake化に伴い使用されていません。

:::details configuration.nix

```nix:configuration.nix
# Edit this configuration file to define what should be installed on
# your system. Help is available in the configuration.nix(5) man page, on
# https://search.nixos.org/options and in the NixOS manual (`nixos-help`).

{ config, lib, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];
  nix = {
    settings = {
      experimental-features = [ "nix-command" "flakes" ];
    };
  };

  # Use the systemd-boot EFI boot loader.
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  networking.hostName = "yasunori-desktop"; # Define your hostname.
  # Pick only one of the below networking options.
  # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.
  networking.networkmanager.enable = true;  # Easiest to use and most distros use this by default.

  # Set your time zone.
  time.timeZone = "Asia/Tokyo";

  # Configure network proxy if necessary
  # networking.proxy.default = "http://user:password@proxy:port/";
  # networking.proxy.noProxy = "127.0.0.1,localhost,internal.domain";

  # Select internationalisation properties.
  i18n.defaultLocale = "en_US.UTF-8";
  # console = {
  #   font = "Lat2-Terminus16";
  #   keyMap = "us";
  #   useXkbConfig = true; # use xkb.options in tty.
  # };

  # Enable the X11 windowing system.
  # services.xserver.enable = true;


  

  # Configure keymap in X11
  # services.xserver.xkb.layout = "us";
  # services.xserver.xkb.options = "eurosign:e,caps:escape";

  # Enable CUPS to print documents.
  # services.printing.enable = true;

  # Enable sound.
  # hardware.pulseaudio.enable = true;
  # OR
  # services.pipewire = {
  #   enable = true;
  #   pulse.enable = true;
  # };

  # Enable touchpad support (enabled default in most desktopManager).
  # services.libinput.enable = true;

  # Define a user account. Don't forget to set a password with ‘passwd’.
  users.users.yasunori = {
    isNormalUser = true;
    extraGroups = [ "wheel" ]; # Enable ‘sudo’ for the user.
    packages = with pkgs; [];
  };

  # List packages installed in system profile. To search, run:
  # $ nix search wget
  environment.systemPackages = with pkgs; [
    vim # Do not forget to add an editor to edit configuration.nix! The Nano editor is also installed by default.
    git
    curl
    gnumake
  ];

  # Some programs need SUID wrappers, can be configured further or are
  # started in user sessions.
  # programs.mtr.enable = true;
  # programs.gnupg.agent = {
  #   enable = true;
  #   enableSSHSupport = true;
  # };

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  # services.openssh.enable = true;

  # Open ports in the firewall.
  # networking.firewall.allowedTCPPorts = [ ... ];
  # networking.firewall.allowedUDPPorts = [ ... ];
  # Or disable the firewall altogether.
  # networking.firewall.enable = false;

  # Copy the NixOS configuration file and link it from the resulting system
  # (/run/current-system/configuration.nix). This is useful in case you
  # accidentally delete configuration.nix.
  # system.copySystemConfiguration = true;

  # This option defines the first version of NixOS you have installed on this particular machine,
  # and is used to maintain compatibility with application data (e.g. databases) created on older NixOS versions.
  #
  # Most users should NEVER change this value after the initial install, for any reason,
  # even if you've upgraded your system to a new NixOS release.
  #
  # This value does NOT affect the Nixpkgs version your packages and OS are pulled from,
  # so changing it will NOT upgrade your system - see https://nixos.org/manual/nixos/stable/#sec-upgrading for how
  # to actually do that.
  #
  # This value being lower than the current NixOS release does NOT mean your system is
  # out of date, out of support, or vulnerable.
  #
  # Do NOT change this value unless you have manually inspected all the changes it would make to your configuration,
  # and migrated your data accordingly.
  #
  # For more information, see `man configuration.nix` or
  # https://nixos.org/manual/nixos/stable/options#opt-system.stateVersion .
  system.stateVersion = "24.05"; # Did you read the comment?

}

```

:::

::::
<!-- textlint-enable -->

私は1つのファイルに大量の記述がしてある状態が好きではないため、可能な限り分割したい派です。
雰囲気としては、Nixを始めた人の第2ステップといった感じでしょうか。

## `import関数`

よく勘違いされますが、`import`は関数です。
よいですか、`import`はNix言語の組込み関数です。
組込み関数は基本的に`builtins`の`AttrSet`に含まれているのですが、使用頻度が高いため`import`だけで呼び出せるようになっています。

https://nix.dev/manual/nix/2.18/language/builtins#builtins-import

https://zenn.dev/asa1984/books/nix-hands-on/viewer/ch01-02-builtins

詳しくは上記のページの説明をご覧いただければ分かるかと思います。

import関数について簡単に説明しておくと、Nix言語が記述されているファイル読み込み`AttrSet`として使えるようになります。
よくあるユースケースとしては次のように使用できます。

```nix:foo.nix
{
  foo = {
    hoge = "fuga";
  };
}
```

```nix:configuration.nix
{
  hoge = (import ./hoge.nix).foo.hoge;
}
```

`foo.nix`を読み込んで、そのまま`foo.hoge`にアクセスしています。
このとき`configuration.nix`の`hoge`には`"fuga"`という文字列が格納されます。

### `default.nix`

`import関数`は基本Nix言語が記述されたファイルを読み込む物ですが、ディレクトリに`default.nix`というファイルがあった場合は、
インポート時にそのディレクトリを指定すれば`default.nix`を読み込んでくれます。

https://zenn.dev/asa1984/books/nix-introduction/viewer/09-nix-lang#import%E9%96%A2%E6%95%B0

その辺の挙動の説明については、上記のリンクを参照してもらうと分かりやすいです。

私の場合、home-managerの設定で次のような使い方をしています。

https://github.com/yasunori0418/dotfiles/blob/1bff134/flake.nix#L68-L82

https://github.com/yasunori0418/dotfiles/blob/1bff134/home-manager/default.nix

このとき、追う順番として次のようになります。

1. `home-manager/default.nix`に`home-manager.lib.homeManagerConfiguration`という関数に渡す引数部分を定義
1. `flake.nix`内で`home-manager/default.nix`をインポートして、定義しておいたAttrSetを引数に渡す

さらに`default.nix`を使っていろいろしているのですが、蛇足になってしまいまうのでここまでの解説にします。
つまりdefault.nixは共通的な処理を書くときや、インポートしている箇所をシンプルにしたいとき便利ということが伝わればよいです！

## `imports`

https://nixos.wiki/wiki/NixOS_modules

NixOSをカスタムして触っている`configuration.nix`というのはモジュールであり、
[NixOS/nixpkgs](https://github.com/NixOS/nixpkgs)でメンテナンスされているモジュールによって定義された設定項目を触っているだけに過ぎません。
そして`imports`というものは、モジュールの`AttrSet（属性）`の1つでしかありません。

それでは`imports`の役割は何かというと、別のモジュールへのパスを記述することで、そのモジュールによって定義された設定項目を使えるようにするという物になります。
ですが、`imports`はそれだけでは終りません。

### 読み込みファイルのマージ機能

別のファイルに記述した設定をマージしてくれる機能があります。
前述したとおり、`imports`はモジュールの読み込みがメインの機能ではありますが、モジュールによって宣言された設定項目を別のファイルにしても読み込んでくれるのです。

次の2つのファイルを見てみましょう。

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/settings/services/openssh.nix

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/settings/services/tlp.nix

このファイルのパスを`imports`に記述するだけで、部分的に重複する`AttrSet`であってもいい感じにマージしてくれます。
結果として内部では次のようになっています。

```nix:configuration.nix
{
  services = {
    tlp = {
      enable = true;
      settings = {
        # 中略
      };
    };
    openssh = {
      enable = true;
      settings = {
        # 中略
      };
    };
  };
}
```

このとき注意すべき点は、トップレベルのAttributeから順番に記述しないといけません。

```nix:systemd/user/services/ssh-agent.nix
{
  description = "SSH key agent";
  wantedBy = [ "default.target" ];
  serviceConfig = {
    Type = "simple";
    Environment = "SSH_AUTH_SOCK=%t/ssh-agent.socket";
    ExecStart = "${pkgs.openssh}/bin/ssh-agent -D -a $SSH_AUTH_SOCK";
  };
}
```

このようにパスやファイル名と内容からssh-agentのsystemd-unitであることは分かるのですが、このファイルを`imports`に追加してもエラーで読み込んでくれません。

### 関数として宣言したファイルの読み込み

読み込もうとしたファイルが関数として記述されていても、次の引数が暗黙的に読み込み先の関数へ渡されます。

- `config`
- `options`
- `pkgs`
- `modulePath`

これらは公式のwikiで列挙されている物で、NixOSとhome-managerの場合で違います。
`man`で`_module.args`の内容を確認しておきましょう。

- NixOS => `man 5 configuration.nix`
- home-manager => `man 5 home-configuration.nix`

`imports`で指定したファイルが関数として読み込むときに、`_module.args`で宣言されている`AttrSet`を引数に渡してくれます。
ただ、各ファイルに引数を全部列挙するのは手間ですし、LSPを使っていると使用していない引数が診断に引っ掛って邪魔です。

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/settings/fonts.nix

ちょうどフォントの設定がそれを回避しています。
フォントの設定に使用しているのは`pkgs`だけですが、それ以外の引数は`...`というものです。

https://zenn.dev/asa1984/books/nix-hands-on/viewer/ch01-01-nix-lang-basics#%E4%BD%99%E5%88%86%E3%81%AAattribute%E3%82%92%E7%84%A1%E8%A6%96%E3%81%99%E3%82%8B

詳しくは上記の内容をご覧頂ければ分かりますが、`pkgs`だけを使用してそれ以外の`AttrSet`は無視するということを`...`はしています。
`imports`に追加するファイルが関数の場合は、引数には`...`のセットが必須といえるでしょう。

### `AttrSet`もマージしてくれる？！

ここまでの説明で`imports`に記述できるのはファイルパスしか渡せないと思いますが、実は`AttrSet`も含めて大丈夫なのです。

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/Desktop/configuration.nix#L20

その証拠に上記のように`import関数`を使ってファイルを読み込み`AttrSet`になった物を`imports`に入れても読み込んでいるのです！
…とは言っても、公式のWikiにそんな説明が無いので、vim-jpの`#tech-nix`で呟いていたところ、
nixpkgsコミッターの[natsukium氏](https://github.com/natsukium)からの天の声を頂きました。

`AttrSet`を読み込んでマージしてくれる謎を教えてくれました！

https://github.com/NixOS/nixpkgs/blob/5c7a370a208d93d458193fc05ed84ced0ba7f387/lib/modules.nix#L181-L191

…おっと、Nix力が強すぎて何が書いてあるか分からないかもしれませんが、
ファイルパスを渡しても最終的に`import関数`を使っているし、`AttrSet`でも読み込んで最終的にマージする処理をしてくれている…ようです。

### 補足（home-managerについて）

`imports`の最初で説明したように、`imports`自体はNixOSのモジュールとしての機能になります。
しかしhome-managerでも同じことができていますが、これはどういうことでしょうか。

実はhome-manager自体もNixOSのモジュールシステムを利用して作られているため、同じ文法で設定の定義が可能になっています。

https://github.com/nix-community/home-manager/blob/a9c9cc6e50f7cbd2d58ccb1cd46a1e06e9e445ff/modules/default.nix#L26-L30

https://github.com/NixOS/nixpkgs/blob/4c8203436897b2d6de8091e65b0c02d962c6e508/lib/modules.nix#L77C71-L85

詳しい理由に関しては、上記のコードを見ることで分かるそう…ですが、またNix力が強くて難しいと感じてしまうかもしれません。
とりあえず、NixOSのモジュールシステムを利用しているということが分かれば大丈夫かと思います。

## まとめ

`import関数`と`imports`に関しては、私もNixOSを触り出して謎に思った部分でした。
現在は`import関数`と`imports`を使いこなしているお陰で、設定ファイルを分割捗っています。

思った以上に難しい内容でしたが、これが皆さんの良きNixライフに繋がれば幸いです！

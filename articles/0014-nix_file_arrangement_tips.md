---
title: "Nixファイルの整理方法"
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
この記事ではNixOSやhome-managerといった物に触れるということはせず、ファイル分割や設定のまとめ方を共有していきます。

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

## `imports`と`import関数`

Nix言語に触れていると最初に遭遇する紛らわしい物です。
この2つの違いと活用方法を見ていきましょう。

### `imports`

https://nixos.wiki/wiki/NixOS_modules

NixOSをカスタムして触っている`configuration.nix`というのはモジュールであり、
[NixOS/nixpkgs](https://github.com/NixOS/nixpkgs)でメンテナンスされているモジュールによって定義された設定項目を触っているだけに過ぎません。
そして`imports`というものは、モジュールの`AttrSet（属性）`の1つでしかありません。

それでは`imports`の役割は何かというと、別のモジュールへのパスを記述することで、そのモジュールによって定義された設定項目を使えるようにするという物になります。
ですが、`imports`はそれだけでは終りません。

#### 読み込みファイルのマージ機能

別のファイルに記述した設定をマージしてくれる機能があります。
前述したとおり、`imports`はモジュールの読み込みがメインの機能ではありますが、モジュールによって宣言された設定項目を別のファイルにしても読み込んでくれるのです。

次の2つのファイルを見てみましょう。

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/settings/systemd/ssh-agent.nix

https://github.com/yasunori0418/dotfiles/blob/1bff134/nixos/settings/systemd/polkit-kde-agent.nix

このファイルのパスを`imports`に記述するだけで、重複するAttrSetであってもいい感じにマージしてくれます。
結果として内部では次のようになっています。

```nix:configuration.nix
{
  systemd.user.services = {
    polkit-kde-authentication-agent = {
      description = "polkit authentication kde agent";
      wantedBy = [ "graphical-session.target" ];
      wants = [ "graphical-session.target" ];
      after = [ "graphical-session.target" ];
      serviceConfig = {
        Type = "simple";
        ExecStart = "${pkgs.libsForQt5.polkit-kde-agent}/libexec/polkit-kde-authentication-agent-1";
        Restart = "on-failure";
        RestartSec = 1;
        TimeoutStopSec = 10;
      };
    };
    ssh-agent = {
      description = "SSH key agent";
      wantedBy = [ "default.target" ];
      serviceConfig = {
        Type = "simple";
        Environment = "SSH_AUTH_SOCK=%t/ssh-agent.socket";
        ExecStart = "${pkgs.openssh}/bin/ssh-agent -D -a $SSH_AUTH_SOCK";
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

#### 関数として宣言したファイルの読み込み

また読み込もうとしたファイルが関数として記述されていても、次の引数が暗黙的に読み込み先の関数へ渡されます。

- `config`
- `options`
- `pkgs`
- `modulePath`

これらは公式のwikiで列挙されている物で、NixOSとhome-managerの場合で違います。
`man`で`_module.args`の内容を確認しておきましょう。

- NixOS => `man 5 configuration.nix`
- home-manager => `man 5 home-configuration.nix`

#### AttrSetもマージしてくれる？！

### `import関数`

## `default.nix`

## `++`や`//`というマージ機能の活用

## 範囲を限定したlet式の活用

## まとめ

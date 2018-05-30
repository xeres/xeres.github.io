---
title: "Visual Studio Code for Linux を WSL 上の Ubuntu 16.04 LTS にインストールする"
date: 2018-05-30 23:19:00 +0900
---
## WSL を有効にする → Ubuntu 16.04 LTS をインストールする

基本的には[公式ブログ][whatsnew-wsl-fcu]の手順に沿って導入すれば問題ない。
Windows 10 バージョン 1709 (Fall Creators Update) 以降、開発者モードは不要となっているし、
`lxrun.exe /install` を叩く手順も非推奨となっている。

以降の手順は Ubuntu 16.04 LTS で動作を確認した。

本当は Ubuntu 18.04 LTS を使いたかったのだが、X Imput Method がうまく動作せず、
Visual Studio Code (VS Code) を起動すると X サーバーごと応答なしになってしまう。
要調査。

## Windows 側に X サーバーをインストールする

[VcXsrv][vcxsrv]とか適当に入れてちょうだい。

## VS Code をインストールする

基本的には[公式ドキュメント][installing-vscode]を参照でよいが、一部の修正が必要。

1. `sudo mv` を利用した手順だと `/etc/apt/trusted.gpg.d/microsoft.gpg` のオーナーが一般ユーザーになってしまうので `sudo cp` を利用する
2. 依存しているが不足しているライブラリがあるので `libgtk2.0-0`, `libxss1`, `libasound2` をインストールする

最後に `DISPLAY` 環境変数を設定して起動すれば問題なし。一旦 VS Code を閉じる。
Windows 側で X サーバーの起動を忘れないように。

```shell
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
rm microsoft.gpg

sudo apt update

sudo apt install -y lib{gtk2.0-0,xss1,asound2}
sudo apt install -y code

export DISPLAY=":0.0"
code
```

## 日本語化を実施する

次に日本語化を行う。

フォントは Google がオープンソースで提供している [Google Noto Fonts][noto] を
利用しているが、`fontconfig` で制御されているので好きなものでよい。

```shell
sudo apt install -y language-pack-ja
sudo apt install -y fonts-noto-cjk

export LANG="ja_JP.UTF-8"
export LC_ALL="$LANG"
```

`export LC_ALL` した際にロケールが正常に反映できない旨のエラーが出た場合、
一度ターミナルを再起動すると良い。

```shell
export DISPLAY=":0.0"
export LANG="ja_JP.UTF-8"
export LC_ALL="$LANG"
code
```

これで日本語化されたメニューや Welcome ページが表示される。
この時点ではまだ日本語入力はできない。

## 日本語入力を有効にする

`uim-mozc` とかも試したのだが上手く動作させることができず、妥協して `uim-anthy` となった。
これで日本語入力が可能となった。

```shell
sudo apt install -y uim{,-{xim,anthy}}

export XIM="uim"
export XMODIFIERS="@im=$XIM"
export GTK_IM_MODULE="$XIM"
export QT_IM_MODULE="$XIM"
export UIM_CANDWIN_PROG="uim-candwin-gtk"

uim-xim &
code
```

日本語入力自体の設定を変えたい場合は `uim-pref-gtk` で普通に設定できる。

## 全角/半角キーを押し下げただけで連打された状態になる不具合に対処する (オプション)

なんか割と有名な事例らしいのだが、`xset -r 49` で修正できるようだ。

```shell
sudo apt install -y x11-xserver-utils

xset -r 49
```

## `~/.profile` に環境変数や起動時コマンドを追記する

ざっとこんな感じ。

`$SHLVL` を見て、`uim-xim` や `xset` が不要に実行されないようにしている。

また、どうも WSL は正常なログインプロセスを経ていないようで `/etc/login.defs` の
`UMASK` が無視されるようなので、ついでに `umask 022` をコメントアウトするとよい。

```bash
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
umask 022 # ついでにコメントアウト

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin directories
PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# ---- 以降追記 ----
export DISPLAY=":0.0"

export LANG="ja_JP.UTF-8"
export LC_ALL="$LANG"

export XIM="uim"
export XMODIFIERS="@im=$XIM"
export GTK_IM_MODULE="$XIM"
export QT_IM_MODULE="$XIM"
export UIM_CANDWIN_PROG="uim-candwin-gtk"

if [ $SHLVL -eq 1 ]; then
    uim-xim >/dev/null &
    xset -r 49
fi
```

[whatsnew-wsl-fcu]: https://blogs.msdn.microsoft.com/commandline/2017/10/11/whats-new-in-wsl-in-windows-10-fall-creators-update/
[vcxsrv]: https://sourceforge.net/projects/vcxsrv/
[installing-vscode]: https://code.visualstudio.com/docs/setup/linux
[noto]: https://www.google.com/get/noto/

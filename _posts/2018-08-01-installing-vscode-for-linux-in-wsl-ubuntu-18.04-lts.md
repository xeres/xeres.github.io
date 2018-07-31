---
title: "Visual Studio Code for Linux を WSL 上の Ubuntu 18.04 LTS にインストールする"
date: 2018-08-01 02:00:00 +0900
---
## WSL を有効にする → Ubuntu 18.04 LTS をインストールする

基本的には[公式ブログ][whatsnew-wsl-fcu]の手順に沿って導入すれば問題ない。
Windows 10 バージョン 1709 (Fall Creators Update) 以降、開発者モードは不要となっているし、
`lxrun.exe /install` を叩く手順も非推奨となっている。

以降の手順は Ubuntu 18.04 LTS で動作を確認した。

## Windows 側に X サーバーをインストールする

[VcXsrv][vcxsrv]とか適当に入れてちょうだい。

## VS Code をインストールする

残念ながら[公式ドキュメント][installing-vscode]通りにやると上手くいかなかった。
上手くいっている人もいるみたいなのだが、なぜか手元の環境では日本語化後に
VS Code が応答なしになってしまう現象が発生して2ヶ月くらいハマっていた。

`code` を `--no-install-recommends` で入れると大丈夫っぽい。

```shell
curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

sudo apt update
sudo apt upgrade -y

sudo apt install -y code --no-install-recommends
sudo apt install -y lib{xss1,gtk2.0-0,asound2,x11-xcb1,xtst6}

# ついでにデフォルトの umask を変更しておく
sed -i -e '/^#umask/s/^#//g' ~/.profile

echo 'export DISPLAY="localhost:0.0"' >> ~/.profile
echo 'export LIBGL_ALWAYS_INDIRECT="1"' >> ~/.profile

source ~/.profile

code
```

この時点では英語版で起動する。当然、日本語入力もできない。

## 日本語化を実施する

次に日本語化を行う。

フォントは Google がオープンソースで提供している [Google Noto Fonts][noto] を
利用しているが、`fontconfig` で制御されているので好きなものでよい。

ちょっと調べた感じだと、日本語環境では `fonts-noto-cjk` があれば良い。
絵文字が必要な場合は `fonts-noto-color-emoji` を追加で入れる。
普通・太字以外の weight の文字が必要な場合は `fonts-noto-cjk-extra` を追加で入れる。
`fonts-noto-hinted` には CJK フォントが含まれていないので、日本語圏では入れるだけ無駄。

```shell
sudo apt install -y fonts-noto-{cjk{,-extra},color-emoji}

sudo apt install -y language-pack-ja

echo 'export LANG="ja_JP.UTF-8"' >> ~/.profile
echo 'export LC_ALL="$LANG"' >> ~/.profile

source ~/.profile
```

システムのデフォルトロケールを日本語にしてしまってもいい場合には、
`~/.profile` に環境変数を追記するのではなく、
`update-locale LANG=ja_JP.UTF-8` すると良い。

これで VS Code が日本語化され、メニューや Welcome ページが日本語で表示されるようになる。
この時点ではまだ日本語入力はできない。

## 日本語入力を有効にする

`fcitx-mozc` が動作した。

```shell
sudo apt install -y dbus-x11
sudo sh -c 'dbus-uuidgen > /var/lib/dbus/machine-id'

sudo apt install -y fcitx-mozc

echo 'export XIM="fcitx"' >> ~/.profile
echo 'export GTK_IM_MODULE="$XIM"' >> ~/.profile
echo 'export QT_IM_MODULE="$XIM"' >> ~/.profile
echo 'export XMODIFIERS="@im=$XIM"' >> ~/.profile
echo 'export DefaultIMModule="$XIM"' >> ~/.profile

source ~/.profile

fcitx-autostart
```

## `fcitx` の設定を変更する

`fcitx-autostart` した状態で下記のコマンドを使用する。

```shell
fcitx-config-gtk3
```

## `mozc` の設定を変更する

`fcitx-autostart` した状態で下記のコマンドを使用する。

```shell
/usr/lib/mozc/mozc_tool --mode=config_dialog
```

## 全角/半角キーを押し下げただけで連打された状態になる不具合に対処する (オプション)

手元の環境では再現率100%だったので対処。`xset -r 49` で修正できるようだ。

```shell
sudo apt install -y x11-xserver-utils

xset -r 49
```

## タイムゾーンを JST にする

デフォルトのタイムゾーンは DST になっているので JST にする。

```shell
sudo sh -c 'echo Asia/Tokyo > /etc/timezone'
sudo dpkg-reconfigure --frontend noninteractive tzdata
```

## OS起動時にやること

1. Xサーバーの起動
2. 下記のコマンドを入力する

```shell
sudo sh -c 'dbus-uuidgen > /var/lib/dbus/machine-id'
fcitx-autostart
xset -r 49
```

これで日本語化＋日本語入力可能な VS Code が使える状態になる。

`dbus-uuidgen --ensure` じゃダメなのかと思ったが、2回目以降の起動で上手くいかなかった。

[whatsnew-wsl-fcu]: https://blogs.msdn.microsoft.com/commandline/2017/10/11/whats-new-in-wsl-in-windows-10-fall-creators-update/
[vcxsrv]: https://sourceforge.net/projects/vcxsrv/
[installing-vscode]: https://code.visualstudio.com/docs/setup/linux
[noto]: https://www.google.com/get/noto/

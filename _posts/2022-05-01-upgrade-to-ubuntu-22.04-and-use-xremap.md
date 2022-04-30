---
title: xremap に切り替えた
---

Ubuntu 22.04 LTS が来ていたので 21.10 から早速アップグレードした。

Ubuntu 21.04 で Wayland セッションがデフォルトになったが、キーリマップのために使っていた [xkeysnail](https://github.com/mooz/xkeysnail) が
当時は Wayland をサポートしていなかったこともあり、引き続き Xorg (GNOME) セッションを使っていた。

22.04 へのアップグレードの機会に Wayland への切り替えを試みたものの、xkeysnail は依然として Wayland をサポートしていない。
xkeysnail の Wayland 対応方法を調べていたところ、Wayland もサポートする [xremap](https://github.com/k0kubun/xremap) を見つけ、切り替えを進めることにした。

xremap については、[GitHub リポジトリ](https://github.com/k0kubun/xremap) に全てまとまっている。
また、作者の方の [ブログエントリー](https://k0kubun.hatenablog.com/entry/xremap) は、目的や背景、Wayland サポートについても丁寧に説明されている。目を通しておくと良いと思う。

使い方は非常にわかりやすい。Alt キーによる IME の切り替えも簡単で、YAML での設定の書き心地も非常に良かった。

以下、一部設定内容をまとめておく。基本的に xkeysnail の設定をベースにしている。

関連ファイルとディレクトリ構造
```
$HOME/.config/
  |- xremap/
  |    |- config.yml
  |    |- start.sh
  |    `- stop.sh
  |
  |- autostart/
  :    |- keymapper.desktop
```

config.yml
```yaml

modmap:
  - name: Global
    remap:
      Capslock: Ctrl_L

  - name: IME
    remap:
      Alt_L:
        held: Alt_L
        alone: Muhenkan
      Alt_R:
        held: Alt_R
        alone: Henkan

keymap:
  - name: Global
    remap:
      # File
      M-s: C-s
      M-o: C-o
# ...
```

起動スクリプト start.sh
```sh
#!/usr/bin/env bash

exec 1> /dev/null 2>> $HOME/.config/xremap/error.log

xhost +SI:localuser:root
sudo xremap --watch=device $HOME/.config/xremap/config.yml &
```

終了スクリプト stop.sh
```sh
#!/usr/bin/env bash

PID=`ps --no-heading -C xremap -o pid | tr -d ' '`

if [ -n "$PID" ]; then
  sudo kill $PID
  echo "Stopped xremap to kill $PID"
fi
```

パスワードなしで sudo できるようにする
```
# /etc/sudoers.d/xremap
hidakatsuya ALL = NOPASSWD: /usr/local/bin/xremap
```
参考: [Man page of SUDOERS](https://linuxjm.osdn.jp/html/sudo/man5/sudoers.5.html)

Startup Application に以下を登録して自動起動
```
[Desktop Entry]
Type=Application
Version=1.0
Name=xremap
GenericName=Keymapper
Exec=/home/hidakatsuya/.config/xremap/start.sh
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
Comment=Enable custom keymapping with xremap
Name[en_US]=xremap
Comment[en_US]=Enable custom keymapping with xremap
```

あっさり移行できてしまい、今の所動作も全く問題ない。本当に素晴らしい。感謝。

ただ、肝心の Wayland への切り替えはまだなので、気持ちが高まってきたらやろうと思う。

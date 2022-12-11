---
title: Ubuntu 22.04 のセッションを X.org から Wayland に移行した
---

重い腰を上げて、メインマシン (ThinkPad X1 Carbon) の Ubuntu 22.04 を Wayland セッションに移行した。

移行に伴い、主に xremap と Ulauncher の対応が必要だった。

## xremap の対応

まず、xremap の README の手順に従い、GNOME Wayland 版をインストールした。
GNOME Shell extension は、ブラウザ拡張ではなく、[Extension Manager](https://github.com/mjakeman/extension-manager) で有効化した。
（おそらく、[GNOME Shell の不具合](https://askubuntu.com/questions/1418937/your-native-host-connector-do-not-support-following-apis-v6) が原因でインストールが不安定だったため）

しばらく使ってみたところ、以下の問題が見つかった。

- Gnome Terminal の設定が適用されていない
- Files (Nautilus) や Gnome Terminal で一部の設定が動作しない

### Gnome Terminal への設定が適用されていない

Gnome Terminal の `WM_CLASS` が X11 と異なるため。再度 Gnome Terminal の `WM_CLASS` を取得して変更した。

```diff
   name: Terminal
     application:
-      only: Gnome-terminal
+      only: gnome-terminal-server
     remap:
       M-t: C-Shift-t
       M-n: C-Shift-n
```

`WM_CLASS` は、xremap の README に記載されているコマンドで取得できる。
```
busctl --user call org.gnome.Shell /com/k0kubun/Xremap com.k0kubun.Xremap WMClass
```

ただし、このコマンドは [実行時にフォーカスがあるウィンドウのクラス名を返す](https://github.com/xremap/xremap-gnome/blob/9a2cc17b2c8288fd8e3ef0c8d8d1bcda45682541/extension.js#L46-L49) ため、次のように `sleep` を挟んでコマンドを遅延実行することで取得した。
```
sleep 3 && busctl --user call org.gnome.Shell /com/k0kubun/Xremap com.k0kubun.Xremap WMClass
```

### Files (Nautilus) や Gnome Terminal で一部動作しない

おそらく https://github.com/k0kubun/xremap/issues/179 によるもの。issue で追加されたワークアラウンドの `keypress_delay_ms` を設定することで対応できた。

```diff
+# Workaround for https://github.com/k0kubun/xremap/issues/179
+keypress_delay_ms: 20
+
 modmap:
```

動作しなかったのは、例えば以下のような設定。
```yaml
  - name: Nautilus
    application:
      only: Org.gnome.Nautilus
    remap:
      M-t: C-t
      M-w: C-w
      M-f: C-f
```

## Ulauncher の対応

`Ctrl+Space` に割り当てた Ulauncher が動作しなかった。

> https://github.com/Ulauncher/Ulauncher/wiki/Hotkey-In-Wayland
> Ulauncher in Wayland does not receive hotkey events when triggered from some windows (like terminal or OS Settings).
 
上記記事の通り、Wayland で利用するには設定が必要とのこと。

記事の手順によって引き続き使うこともできるが、Ulauncher をやめ、GNOME の Activities を `Ctrl+Space` に割り当てて使ってみることにした。
しっくりこなければ、Ulauncher か別の選択肢を検討することにする。


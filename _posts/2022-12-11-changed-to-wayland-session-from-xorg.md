---
title: Ubuntu 22.04 のセッションを X.org から Wayland に移行した
---

重い腰を上げて、メインマシン (ThinkPad X1 Carbon) の Ubuntu 22.04 を Wayland セッションに移行した。

インストールは、基本的に xremap の README の手順通り。
GNOME Shell extension は、ブラウザ拡張ではなく、[Extension Manager](https://github.com/mjakeman/extension-manager) でインストールした。
（GNOME Shell の不具合によりインストールが不安定だったため。
詳細は [こちら](https://askubuntu.com/questions/1418937/your-native-host-connector-do-not-support-following-apis-v6) を参照）

しばらく使ってみたところ、以下の問題が見つかった。

- Gnome Terminal への設定が適用されていない
- Files (Nautilus) や Gnome Terminal で一部動作しない
- Ulauncher は Wayland にネイティブでは対応していない

## Gnome Terminal への設定が適用されていない

`WM_CLASS` が異なることが原因と思われる。再度 Gnome Terminal の `WM_CLASS` を取得して変更。

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

ただし、このコマンドは [実行時にアクティブなフォーカスがあるウィンドウのクラス名を返す](https://github.com/xremap/xremap-gnome/blob/9a2cc17b2c8288fd8e3ef0c8d8d1bcda45682541/extension.js#L46-L49) ため、次のように `sleep` を挟んでコマンドを遅延実行することで取得した。
```
sleep 3 && busctl --user call org.gnome.Shell /com/k0kubun/Xremap com.k0kubun.Xremap WMClass
```

## Files (Nautilus) や Gnome Terminal で一部動作しない

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

## Ulauncher は Wayland にネイティブでは対応していない

> https://github.com/Ulauncher/Ulauncher/wiki/Hotkey-In-Wayland
> Ulauncher in Wayland does not receive hotkey events when triggered from some windows (like terminal or OS Settings).

上記の対応手順で引き続き使うことができるが、Ulauncher をやめて、GNOME の Activities を `Ctrl+Space` に割り当てて使ってみることにした。
しっくりこなければ Ulauncher か別の選択肢を検討することにする。


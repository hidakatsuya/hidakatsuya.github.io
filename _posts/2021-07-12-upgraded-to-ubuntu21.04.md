---
layout: post
title: Ubuntu 21.04 にアップグレードした
---

先日 Ubuntu 21.04 がリリースされた。

https://discourse.ubuntu.com/t/hirsute-hippo-release-notes/19221

現在使っている 20.10 のサポートは 7/22 までということで、重い腰を上げてアップグレードすることにした。
アップグレード自体は、特段大きな問題もなく完了。

21.04 の大きな特徴の一つは、Wayland セッションのデフォルト化だろう。

> Wayland is now the default on most configurations, which features better security and performance

これまで Xセッション(GNOME on Xorg) を使ってきたが、この機会に Wayland への移行を試みた。

が、移行を断念した。

理由は、キーリマッピングツールの [xkeysnail](https://github.com/mooz/xkeysnail) が動作しなかったため。xkeysnail を実行すると次のエラーが発生する。

```
ImportError: sys.meta_path is None, Python is likely shutting down
```

詳しく調べていないが、以下の issue に類する。

- https://github.com/mooz/xkeysnail/issues/135
- https://github.com/mooz/xkeysnail/issues/125

それ以外は問題なく動作していそうだった。以前(Ubuntu18 のころかな?)に試したときのモッサリ感もなく、xkeysnail の問題さえ解決すれば移行したいと思っている。

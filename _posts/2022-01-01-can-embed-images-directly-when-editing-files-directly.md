---
title: GitHub上で直接ファイルを作成/編集するときでも、画像を直接を埋め込めるようになっていた
date: 2022-01-01 22:20:00 +0900
---

普段記事を投稿するときは、github.com 上から直接作成している。

![image](https://user-images.githubusercontent.com/739339/147850927-8e0ee231-e641-4a26-8614-39406f174fd7.png)

これはすごく便利なのだが、pull request や issue などのように、D&D やペーストなどによって画像を直接埋め込むことができない（と思っていた）。
そのため、画像を埋め込む場合は、事前に `images` ディレクトリに画像をアップロードし、記事中からそのファイルを参照するような運用にしていた。
正直に言ってこれはとても面倒だった。

がしかし、2022年1月1日、いつものように記事を作成していたとき、編集フォーム下部にふと目が行った。

> Attach files by dragging & dropping, selecting or pasting them.

![image](https://user-images.githubusercontent.com/739339/147851136-578d1986-95f5-440f-8db6-48744a5e2fe3.png)

...え？

実際にやってみると、普通に画像を選択してアップロードできるし、画像をペーストして、pull request や issue などと同様、カーソルがある位置に画像が挿入することもできる。

...便利すぎる

同日、[サイトのテーマを Primer Theme に変更した](2022-01-01-change-theme-of-site-to-primer.md)
にて、サイトのスタイルを GitHub 謹製の Primer テーマに変更したので、github.com 上のファイルのレビュー結果と実際の記事の結果がより近くなったこともあり、
これで記事の管理は github.com 上で完全にできるようになった。

なお、いつからこれができるようになったかまでは調べていないが、以前はできなかったのは確か。

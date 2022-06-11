---
title: GitHub Pages (Jekyll) でコードブロック中の中括弧が描画されない
date: 2022-06-11 21:33:00 +0900
---

[GitHub Actions での Docker によるテストの高速化を試みた](2022-06-11-try-to-make-github-action-faster.md)
の記事を公開した後、コードブロック内の {% raw %} `${{ ... }}` {% endraw %} が全て消えていることに気づいた。

![a](https://user-images.githubusercontent.com/739339/173188286-2bc9b973-cb60-451a-976c-069389f3a160.png)

おかしい。正しくはこう。

![b](https://user-images.githubusercontent.com/739339/173188459-cfc14e67-431d-4e75-abdb-acabee4a5974.png)


手元で `jekyll serve` してみると確かに警告が出力されていた。

{% raw %}

```
Liquid Warning: Liquid syntax error (line 139): Expected end_of_string but found open_round in "{{ hashFiles('Gemfile.lock') }}" in /home/hidakatsuya/github/hidakatsuya.github.io/_posts/2022-06-11-try-to-make-github-action-faster.md
```

{% endraw %}

## 解決方法

下記公式ドキュメントに書いてあるように、コンテンツに連続する中括弧が含まれる場合は [Liquid](https://github.com/Shopify/liquid) の描画を無効にする必要がある。

http://jekyllrb-ja.github.io/docs/liquid/tags/

無効にする方法は、Jekyll のバージョンによって異なるが、執筆時点の GitHub Pages での Jekyll 3.9.2 では、

{% gist eac869c02647983bbbdbc92205ee5bed github-pages-post.md.txt %}

のようにすれば良い。冒頭の記事の実際の修正コミットは [24da40e](https://github.com/hidakatsuya/hidakatsuya.github.io/commit/24da40e7779ac5581f4305c3a8fbf072a8c73a50)。

なお、Jekyll 4.0 以降では、単にフロントマターに `render_with_lizuid: false` を書くことで Liquid の描画を無効化できる。

## 余談

ちなみに、GitHub Pages で使われている Jekyll のバージョンやプラグインの種類とバージョンは公式の以下のページで確認できる。

https://pages.github.com/versions/

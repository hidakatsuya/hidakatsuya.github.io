---
layout: post
title: Markdown の URL 自動リンクと CommonMark と GitHub Flavored Markdown
---

Markdown は、今や書かない日はないと言って良いほど身近なものだ。現にこの記事も Markdown で書いている。
しかし、自分が書いている Markdown がどんなものなのかを深く考えたり調べることはなかった。

そんな中、このサイトの記事を書いている時「Markdown 中の URL に自動的にリンク貼るにはどうすればいいんだっけ」という疑問から、
タイトルの CommonMark や GitHub Flavored Markdown、そして Markdown の歴史について浅い知識を得る機会が訪れる。

## GitHub Pages の Markdown 中における URL の自動リンク

GitHub Pages のデフォルトの挙動(後述)では、Markdown 中の URL にはそのままではリンクが貼られない。

```markdown
https://github.com # => plain text のまま
```

しかし、普段使っている GitHub では自動的にリンクが貼られる。これはなぜか。

答えは次の通り。

- GitHub Pages (Jekyll) のデフォルトの Markdown Parser は [kramdown](https://kramdown.gettalong.org/syntax.html) で、kramdown は「前述の例の自動リンク」を少なくとも標準ではサポートしていない
- GitHub (github.com) の Markdown は GitHub Flavored Markdown であり、それは「前述の例の自動リンク」をサポートしている

なお、kramdown でも次のような記述で URL にリンクを貼ることはできる(後で知った)。

```markdown
<https://github.com> # => <a> で括られる
```

[Automatic Links](https://kramdown.gettalong.org/syntax.html)

## GitHub Flavored Markdown

では、GitHub Flavored Markdown とはなんなのか。「github.com で使える Markdown のやつ」ぐらいの認識しかなかったが、
調べてみると、それは概ね正しい回答であることがわかった。

前述の自動リンクを解決する方法を調べた結果、GitHub Pages の `_config.yml` を次のように変更することで実現できることがわかった。

```diff
+ markdown: CommonMarkGhPages
+ commonmark:
+  extensions:
+    - autolink
+    - strikethrough
+    - table
```

[Change the markdown processor to CommonMarkGhPages](https://github.com/hidakatsuya/hidakatsuya.dev/commit/11362c53f8b9c771c00bf118847eb92f1c6206ef)

この設定では、GitHub Pages の Markdown Parser を CommonMarkGhPages に変更し、`autolink` を含む、いくつかの extension を有効にしている。

では、この CommonMarkGhPages とはなんなのか。それを説明するには、まず CommonMark を知る必要がある。

> We propose a standard, unambiguous syntax specification for Markdown, along with a suite of comprehensive tests to validate Markdown implementations against this specification. We believe this is necessary, even essential, for the future of Markdown.  
> https://commonmark.org/

つまり、CommonMark とは「Markdown の標準の明確な仕様と、その仕様を検証するためのテストスイート」ということらしい。
実際に仕様を読むと、エッジケースの仕様について詳しく説明されていることがわかる。

https://spec.commonmark.org/0.29/

そして、この CommonMark から派生した Markdown の仕様が GitHub Flavored Markdown である。

https://github.github.com/gfm/

実際、GitHub Flavored Markdown の parser の実装 [github/cmark-gfm](https://github.com/github/cmark-gfm) は、
[commonmark/cmark](https://github.com/commonmark/cmark) を fork したものになっている。

そして CommonMarkGhPages だが、残念ながらこれについての明確な説明を見つけることができなかったが、
「GitHub Flavored Markdown に準拠した GitHub Pages 用の Markdown Parser」であると推察できる。

ちなみに、GitHub Flavored Markdown の仕様を見ると、CommonMark に加える形で前述の自動リンクの仕様が拡張されていることがわかる。

> 6.9Autolinks (extension)
> GFM enables the autolink extension, where autolinks will be recognised in a greater number of conditions.  
> https://github.github.com/gfm/#autolinks-extension-

## Markdown の歴史

CommonMark の役割に垣間見えるように、Markdown の歴史はなかなか興味深いものがある。この辺りは下記の記事で詳しく、そして読みやすくまとめられている。

[GitHub Flavored Markdown は何であって何でないか](https://qiita.com/tk0miya/items/6b81e0e4563199037018)

## 最後に

実は、これを書くまでは kramdown での自動リンクの記法 `<https://github.com>` を知らなかった。
自動リンクの観点では、kramdown のままでも支障はないが、github.com 上で直接記事の作成/編集を行うことが多いことを踏まえると、
Markdown は GitHub Flavored Markdown に統一しておいた方が良いと考え、Markdown Parser は CommonMarkGhPages に変更した。

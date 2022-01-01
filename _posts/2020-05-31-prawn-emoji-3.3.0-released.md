---
title: Emoji 13.0 に対応した prawn-emoji v3.3.0 をリリースした
---

[prawn-emoji](https://github.com/hidakatsuya/prawn-emoji) の [バージョン 3.3.0](https://github.com/hidakatsuya/prawn-emoji/blob/master/CHANGELOG.md#330) をリリースした。

このリリースで 2020年3月10日にリリースされた [Emoji v13.0](https://unicode.org/emoji/charts-13.0/emoji-released.html) に対応した。
といっても、その対応は [@aried3r さんの Pull request](https://github.com/hidakatsuya/prawn-emoji/pull/33) をマージしただけ。感謝。

余談だけど、`rake release` タスクを初めて使ったが便利だった。これは Bundler の機能で、Rakefile で `bundler/gem_tasks` を require すれば利用できるようになる。
`bundle gem` による gem のスケルトンではデフォルトで利用できるようになっているはず。

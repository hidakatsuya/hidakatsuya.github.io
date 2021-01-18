---
layout: post
title: Ruby3 に対応した Thinreports v0.12.0 をリリースした
date: 2021-01-18 22:16:00 +0900
---

2021-1-16 に Thinreports v0.12.0 をリリースした。

主に、thinreports-generator gem に Ruby3 のサポートを追加するリリースだが、
editor についても一つの機能追加と依存ライブラリの更新が行われている。

## 変更点

generator は、Ruby3 のサポートに加え、Prawn v2.3 と v2.4 のサポートも追加した。Prawn は v2.4 で Ruby3 をサポートしたので、
Ruby3 で generator を使う場合は、必然的に Prawn v2.4 を使うことになる。

また、editor は、Ctrlキーとマウスホイールによるキャンバスの拡大縮小機能をサポートした。

- [公式アナウンス](http://www.thinreports.org/news/2021/01/thinreports-v0_12_0-released/)
- [Changelog (editor)](https://github.com/thinreports/thinreports-editor/blob/master/CHANGELOG.md)
- [Changelog (generator)](https://github.com/thinreports/thinreports-generator/blob/master/CHANGELOG.md)

## generator v0.12.0 におけるパフォーマンス問題

generator は [Prawn](https://github.com/prawnpdf/prawn) という PDFライブラリを使っている。
そして、Prawn は [ttfunk](https://github.com/prawnpdf/ttfunk) という TTF (True Type Font) を扱うライブラリに依存している。

しかし、この ttfunk の v1.6 以降には、 [ttfunk#82](https://github.com/prawnpdf/ttfunk/issues/82) で報告されている通り、
v1.5 に比べてパフォーマンスが大幅に低下するという問題がある。

generator v0.12.0 がこの問題の影響を必ず受けるかというとそうでもない。

少しややこしいが、generator v0.12.0 が依存している Prawn 及び ttfunk、そして Ruby の関係性を下記に示す。

| Prawn | ttfunk | Ruby |
| -- | -- | -- |
| 2.4 | `~> 1.7` | `>= 2.5` |
| 2.3 | `~> 1.6` | `~> 2.5` |
| 2.2 | `~> 1.5` | `~> 2.1` |

上記より、Prawn v2.3 以降を使う場合に、この問題の影響を受けることになる。
そして、Ruby3 で generator を使う場合も、現状ではこの問題の影響を受けることになる。
なぜなら、Ruby3 を使うためには ttfunk v1.7 以降に依存する Prawn v2.4 を使う必要があるためだ。

また、Ruby2 で generator v0.12.0 を使う場合でも注意が必要だ。
確実に回避したいなら、下記の通り、Prawn v2.2 と ttfunk v1.5 に依存をロックしておく必要がある。もしくは、まだ v0.12.0 を使わないのも手だ。

```ruby
# Gemfile
gem 'prawn', '~> 2.2.2'
gem 'ttfunk', '~> 1.5.1'
```

この問題は [ttfunk#PR83](https://github.com/prawnpdf/ttfunk/pull/83) で対応が進んでいるものの、現時点ではリリースされていない。
このパフォーマンスの問題の影響は決して小さくないため、早期に取り込まれるようにコントリビュートしていきたい。

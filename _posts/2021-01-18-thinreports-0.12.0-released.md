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

## Generator v0.12.0 におけるパフォーマンス問題

generator は [Prawn](https://github.com/prawnpdf/prawn) という PDFライブラリを使っている。
そして、Prawn は [ttfunk](https://github.com/prawnpdf/ttfunk) という TTF (True Type Font) を扱うライブラリに依存している。

しかし、この ttfunk の v1.6 以降には、 [ttfunk#82](https://github.com/prawnpdf/ttfunk/issues/82) で報告されている通り、
v1.5 に比べてパフォーマンスが大幅に低下するという問題がある。

generator v0.12.0 はこの問題の影響を受けてしまうのだが、少々複雑な状況になっている。

下記は、generator v0.12.0 が依存している Prawn 及び ttfunk、そして Ruby の関係性をまとめたものだ。

| Prawn | ttfunk | Ruby | パフォーマンス問題の影響 |
| -- | -- | -- | -- |
| 2.4 | `~> 1.7` | `>= 2.5` | 受ける |
| 2.3 | `~> 1.6` | `~> 2.5` | 受ける |
| 2.2 | `~> 1.5` | `~> 2.1` | 受けない |

Prawn v2.3 以降は ttfunk v1.6 に依存しているため、Prawn v2.3 以降を使う場合は影響を受ける。
また、Ruby3 で使う場合も、現状では影響を受けることになる。

ttfunk v1.5 を使うようにすることで影響を避けることができるが、確実に回避するためには、下記のように ttfunk v1.5 に依存をロックしておくのが無難。

```ruby
# Gemfile
gem 'ttfunk', '~> 1.5.1'
```

なお、この問題は [ttfunk#PR83](https://github.com/prawnpdf/ttfunk/pull/83) で対応が進んでいるが、現時点ではリリースされていない。
このパフォーマンスの問題の影響は決して小さくないため、早期に取り込まれるようにコントリビュートしていきたい。

詳細は [thinreports-generator#104](https://github.com/thinreports/thinreports-generator/issues/104) を参照して欲しい。

---
layout: post
title: 一年半ぶりに Thinreports v0.11.0 をリリースして、リリース手順などの運用周りを見直した
---

[Thinreports v0.11.0](http://www.thinreports.org/news/2020/06/thinreports-v0_11_0-released/) をリリースした。
小さな breaking changes も含まれるが、bugfix が中心のリリースである。詳細はリンク先のアナウンスを参照して欲しい。

今回のリリースは、その手順を思い出すことに苦労した。

一年半ぶりのリリースということもあるが、これまでリリースのほぼ全てを自分一人で実施し、
その手順の自動化やドキュメント化などの整備も不十分だった。

そういった背景もあり、良い機会なのでできることから実施した。

#### 他のメンテナとの協働

全ての作業で、他のメンテナを積極的に巻き込んで進めた。

#### 議論とステータスの透明化

全ての作業を pull request や issue で実施し、その作業の全ての議論をそれぞれのコメント欄で行った。

- [Editor - PR#80 Release v0.11.0](https://github.com/thinreports/thinreports-editor/pull/80)
- [Generator - PR#105 Release v0.11.0](https://github.com/thinreports/thinreports-generator/pull/105)

リリース自体の判断やリリースに関する他の議論は `thinreports/thinreports` に `Release v0.11.0` という issue を作成し、
タスクの管理やリリース日程の調整、その他の議論を行うようにした。

[Thinreports - Issue#10 Release v0.11.0](https://github.com/thinreports/thinreports/issues/10)

#### リリース手順の整備

Editor, Generator それぞれのリリース手順を README に記載するとともに、その手順に従って実施した。

- [Releasing Generator](https://github.com/thinreports/thinreports-generator#releasing-generator)
- [Releasing Editor](https://github.com/thinreports/thinreports-editor#releasing-editor)

以上、大変ではあったが、自分一人でリリースするよりも楽しく達成感があった。今後もぜひ続けたい。そして、そのための仕組みやドキュメントの整備を少しづつ行っていく。

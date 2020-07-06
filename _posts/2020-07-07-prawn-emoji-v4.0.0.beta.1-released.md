---
layout: post
title: prawn-emoji v4.0.0.beta.1 をリリースした
---

prawn-emoji の次期メジャーバージョンのベータ版 [v4.0.0.beta.1](https://github.com/hidakatsuya/prawn-emoji/blob/v4.0.0.beta.1/CHANGELOG.md#400beta1) をリリースした。

このバージョンの主な変更点は次の通り:

1. 絵文字の描画方法を刷新し、全角スペースなどの代替文字への置換を行わないようにした
2. Ruby 2.4 のサポート終了

(1) は、[Issue#34: Some emojis are not displayed.](https://github.com/hidakatsuya/prawn-emoji/issues/34) を解決するための対応で、
絵文字の描画方法を根本的に変更したもの。

- v3: テキスト中の絵文字は、全角スペースに置換し、その場所に絵文字画像を描画
- v4: テキストを絵文字で分割し、分割したテキストは絵文字の幅の分だけ位置をずらして描画する。ずらした位置に絵文字画像を描画

Issue#34 は全角スペースを表示できないフォントを使っていることが原因だった。v4 は全角スペースを描画しないため、この問題は解決するはずだ。

また、若干ではあるが、v3 と v4 では絵文字の描画結果に差異があるため、ベータリリースという形を取っている。

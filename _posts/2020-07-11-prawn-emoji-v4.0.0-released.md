---
title: prawn-emoji v4.0.0 をリリースした
---

[ベータ版](/2020/07/07/prawn-emoji-v4.0.0.beta.1-released.html) を経て、[v4.0.0](https://github.com/hidakatsuya/prawn-emoji/blob/master/CHANGELOG.md#400) の正式版をリリースした。

Enhancements
- 新しい絵文字の描画処理に変更したことにより、特定のフォントへの依存性を排除

Breaking Changes
- Ruby 2.4 のサポートを終了
- 絵文字の描画結果が少しだけ変化 (新しい絵文字の描画処理への変更によるもの)

Bug Fixes
- [Some emojis are not displayed #34](https://github.com/hidakatsuya/prawn-emoji/issues/34)

これまでの描画処理には、以前から不満を持っていたので、それが解決できて満足。あとは、test-unit への移行も近いうちにやりたいと思っている。

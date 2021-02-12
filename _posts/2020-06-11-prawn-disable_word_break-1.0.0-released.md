---
layout: post
title: 単語単位の折り返しを無効にする Prawn 拡張 prawn-disable_word_break をリリースした
---

[prawn-disable_word_break gem](https://github.com/hidakatsuya/prawn-disable_word_break) の最初のバージョン 1.0.0 をリリースした。

> Fast, Nimble PDF Writer for Ruby
> https://github.com/prawnpdf/prawn

prawn は非常に使いやすい gem だが、ハイフネーションなど、テキストの単語の折り返しを制御するオプションが存在しない。
他にも、タブや半角スペース、ソフトハイフン、ゼロ幅スペースによる単語折り返しがあり、prawn はこれらがデフォルトで有効になっている。

しかし、これらの単語の折り返しは、日本語環境では不要なケースが多いのではないかと思う。

今回、まさにその場面に遭遇したので、単語折り返しを無効にする拡張を作った。

```ruby
require 'prawn/disable_word_break'

Prawn::Document.generate 'foo.pdf' do
  disable_word_break do
    text_box 'text without word-breaking', at: [100, 100], width: 50, height: 50
  end
end
```

`disable_word_break {}` 内のテキスト描画は単語の折り返しが全て無効化される。具体的な PDF の結果は次の通り。

![](https://raw.githubusercontent.com/hidakatsuya/prawn-disable_word_break/master/doc/comparison-of-word-breaking.png)

詳細は [README.md](https://github.com/hidakatsuya/prawn-disable_word_break/blob/master/README.md) を参照して欲しい。

なお、今回は拡張を作ったが、単語の折り返しを制御するオプションは prawn 本体にあると良いと思っている。今後、Pull request で提案していくつもりだ。

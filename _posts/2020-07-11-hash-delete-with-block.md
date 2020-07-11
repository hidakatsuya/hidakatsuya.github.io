---
layout: post
title: Ruby の Hash#delete はブロックを渡すことができ、これがとても便利だった
---

`Hash#delete` はブロックを受け取ることを初めて知った。そして、これがとても便利だったのでまとめておく。

`Hash#delete` は、

```ruby
h = { a: 'aaa', b: 'bbb' }

h.delete(:a) #=> 'aaa'
h.delete(:nonexistent_key) #=> nil

p h #=> { b: 'bbb' }
```

のように、指定したキーを取り除くことができるメソッドで、

- 取り除いたキーの値を返す
- 指定されたキーが見つからなかった場合 `nil` を返す
- 破壊的に変更する

という特徴を持っている。

そして、ブロックを渡した場合、

```ruby
h.delete(:x) { |key| "#{key} は見つかりませんでした" } #=> ":x は見つかりませんでした。"
```

のように、指定したキーが見つからなかった場合にブロックが評価され、その結果が戻り値となる。

キーが見つからなかった場合の戻り値についてまとめると、

- ブロックを渡さない場合
  - 常に `nil` を返す
- ブロックを渡した場合
  - ブロックを評価した結果を返す

となる。

さて、これの何が便利だったかというと、つい最近 [prawn-emoji のこの部分](https://github.com/hidakatsuya/prawn-emoji/blob/bec36de9b54abfd290dd6e45c6079148ef94dd2e/lib/prawn/emoji/drawable.rb#L8-L18)
の実装をするときに、この仕様のおかげでシンプルに書くことができた。

```ruby
 1: # == Additional Options
 2: # <tt>:emoji</tt>:: <tt>boolean</tt>. Whether or not to draw an emoji [true]
 3: def draw_text!(text, options)
 4:   draw_emoji = options.delete(:emoji) { true }
 5:
 6:  if draw_emoji && Emoji::Drawer.drawable?(text)
 7:    emoji_drawer.draw(text.to_s, options)
 8:  else
 9:    super
10:  end
11: end
```

このコードは、既存の `draw_text!`を拡張する実装である。L#4 で `Hash#delete` を使っており、

```ruby
 4:   draw_emoji = options.delete(:emoji) { true }
```

- `options` の `:emoji` キーを削除し、値(boolean) を `draw_emoji` に格納する
- `:emoji` キーが存在しない場合は `true` を `draw_emoji` に格納する

ということをこの一行で実現することができる。

`:emoji` option の仕様について少し補足しておくと、

1. `:emoji` は拡張した option なので、L#10 の `super` に `options` を渡す際に削除しておきたい
2. `:emoji` はデフォルト `true` にしたい

という背景があった。

そして、元々、この部分(L#4)は、`Hash#fetch(key, default)` を使って実装していたが、(1) の要件を満たすためには、
どうしても別途 `:emoji` キーを消すコードを入れないといけなかった。

```ruby
 1: # == Additional Options
 2: # <tt>:emoji</tt>:: <tt>boolean</tt>. Whether or not to draw an emoji [true]
 3: def draw_text!(text, options)
 4:   draw_emoji = options.fetch(:emoji, true) #=> Hash#fetch を使った場合
 5:   options.delete(:emoji)                   #=> 別途 :emoji キーを削除しないといけない
 6:
 7:  if draw_emoji && Emoji::Drawer.drawable?(text)
 8:    emoji_drawer.draw(text.to_s, options)
 9:  else
10:    super
11:  end
12: end
```

細かいことだが、`Hash#delete` を使うとこれをスッキリ書くことができる (読みやすいかどうかは別)

See also [Ruby2.7.0 リファレンスマニュアル > Hash クラス](https://docs.ruby-lang.org/ja/latest/class/Hash.html#I_DELETE)

ちなみに、

`Hash#delete` の "取り除いたキーの値を返す" という特徴を使って、

```ruby
draw_emoji = options.delete(:emoji) || true
```

という方法も考えられるが、これは正しく動作しない。`emoji: false or nil` のとき、draw_emoji は `true` になってしまうからだ。

---
title: Module#append_features と Module#<
---

ふと `ActiveSupport::Concern` が base クラスにメソッドなどを定義する流れが気になってコードを読んでいると、いくつか新しい発見があった。

## Module#append_features

`ActiveSupport::Concern` は読んでみると `append_features` メソッドに辿り着く。
https://github.com/rails/rails/blob/912096d4ce930b8e7e5d91e0c86bae2091fda0e4/activesupport/lib/active_support/concern.rb#L129

このメソッドの下記の箇所が該当すると思われたが、`append_features` メソッドをどこでコールしているかがわからない。
```ruby
base.class_eval(&@_included_block)
```
https://github.com/rails/rails/blob/912096d4ce930b8e7e5d91e0c86bae2091fda0e4/activesupport/lib/active_support/concern.rb#LL138C29-L138C29

で、調べてみると、どうやらこれは Ruby のメソッドらしい。新しいメソッドではなく、少なくとも Ruby 2.0 からある。

https://docs.ruby-lang.org/ja/latest/method/Module/i/append_features.html
> モジュール(あるいはクラス)に self の機能を追加します。
> このメソッドは Module#include の実体であり、

試してみたところ、確かに `Module#include` の実体っぽい。

```
$ irb
irb(main):001:1* module M
irb(main):002:2*   def self.append_features(base)
irb(main):003:3*     base.class_eval do
irb(main):004:4*       def m_method
irb(main):005:4*         puts 'm_method'
irb(main):006:3*       end
irb(main):007:2*     end
irb(main):008:1*   end
irb(main):009:0> end
=> :append_features
irb(main):010:1* class Foo
irb(main):011:1*   include M
irb(main):012:0> end
=> Foo
irb(main):013:0> Foo.new.m_method
m_method
```

`Module#include` が評価されるときに、`append_features` が呼ばれている。

```
$ irb
irb(main):001:1* module M
irb(main):002:2*   def self.append_features(base)
irb(main):003:2*     puts 'append_features called'
irb(main):004:2*     super
irb(main):005:1*   end
irb(main):006:0> end
=> :append_features
irb(main):007:1* class Foo
irb(main):008:1*   include M
irb(main):009:0> end
append_features called
=> Foo
```

なるほどなぁ。

## Module#<

同じく、以下のコードが気になった。
```ruby
return false if base < self
```
https://github.com/rails/rails/blob/912096d4ce930b8e7e5d91e0c86bae2091fda0e4/activesupport/lib/active_support/concern.rb#L134

予想した通りだったが、継承関係を調べる演算子だった。

> 比較演算子。self が other の子孫である場合、 true を返します。 self が other の先祖か同一のクラス／モジュールである場合、false を返します。

https://docs.ruby-lang.org/ja/latest/method/Module/i/=3c.html

```
$ irb
irb(main):001:1* module M
irb(main):002:0> end
=> nil
irb(main):003:1* class Foo
irb(main):004:1*   include M
irb(main):005:0> end
=> Foo
irb(main):006:1* class Bar < Foo
irb(main):007:0> end
=> nil
irb(main):008:0> Foo < M
=> true
irb(main):009:0> Bar < Foo
=> true
irb(main):010:0> Bar < M
=> true
irb(main):011:0> M < Bar
=> false
```

面白い。

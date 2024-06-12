---
title: Redmined CLI のサブディレクトリでの実行をサポートしたことで、プラグインの開発が快適になった
---

Redmine やプラグインは、基本的にこの Redmined CLI を使って開発している。

https://github.com/hidakatsuya/redmined

しかし、プラグインの開発は面倒な点が多い。

例えば、`redmine_hello_world` を開発する場合、次のような手順で行う。

VSCode で `plugins/redmine_hello_world` を開く。デフォルトではターミナルはプラグインのディレクトリ `plugins/redmine_hello_world` で開くため、
Redmine を起動するためには、わざわざルートディレクトリに移動しなければならない。

```
$ cd ../../
$ redmined bin/rails s
```

これを解消するために、`redmined` をサブディレクトリで実行できるように対応した。
https://github.com/hidakatsuya/redmined/commit/7e2f08b38544c24a44e4640716976c89b19584c6

これによって、プラグインのディレクトリから移動することなく、テストの実行や Redmine の起動を行うことができるようになった。

```
$ pwd
path/to/redmine/plguins/redmine_hello_world

$ redmined bin/rails redmine:plugins:test
...
```

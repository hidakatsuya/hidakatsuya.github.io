---
layout: post
title: prawn-emoji の test framework を test-unit に移行した
date: 2021-02-13 07:29:00 +0900
---

少し前の話になるが、正月休みの時間を使って、[prawn-emoji](https://github.com/hidakatsuya/prawn-emoji) の test framework を minitest から [test-unit](https://test-unit.github.io/) に変更した。

単純に minitest (spec) の構文を test-unit に変更したぐらいで、特に移行で困ったことはない。以下移行コミット。

[Migrate the test framework to test-unit gem](https://github.com/hidakatsuya/prawn-emoji/commit/287950b96cf04dcc59989a1abe93d75be2d84a2e)

個人的には、ruby の test framework はもう test-unit で良いと思っている。

- 構文や assertion など、シンプルで考えることが少ない
- 標準の出力フォーマットが好み
- `test 'description' do ... end` の構文がしっくりくる
- `sub_test_case` も標準で使える

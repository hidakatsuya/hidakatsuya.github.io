---
layout: post
title: Logicool MX Anywhere 3 マウスに移行した
date: 2020-11-01 15:34:00 +0900
---

10/29 に発売したばかりの [Logicool MX Anywhere 3](https://www.logicool.co.jp/ja-jp/products/mice/mx-anywhere-3.910-006005.html) を買った。

## 背景

それまで使っていたマウスはゲーミングマウスの [Logicool G304](https://gaming.logicool.co.jp/ja-jp/products/gaming-mice/g304-lightspeed-wireless-gaming-mouse.910-005287.html)。
今回のマウスで通算5代目。その前はなんだったかな。エルゴノミクスマウスの安いやつだったと思う。

その G304 は、性能・大きさ・重さ、どれをとっても満足するものだったが、一点だけ不満があった。

自分の作業環境は、主に仕事用 Mac mini とプライベートの Thinkpad X1 Carbon (Ubuntu) の2つがあるが、G304 はマルチデバイスに対応していない。
そのため、キーボード ([Logitech MX Keys US配列](2020-05-16-started-using-mx-keys.md)) も含めると、次のような接続環境になっていた。

| 環境 | G304 (mouse) | MX Keys (keyboard) |
| -- | -- | -- |
| Mac mini | USB レシーバ (Lightspeed) | USB レシーバ (Unifying) |
| Thinkpad | USB レシーバ (Lightspeed) | Bluetooth |

この環境の問題点は次の通り。

1. Mac mini と Thinkpad を切り替えるときに USB レシーバを差し替える必要がある
1. Mac mini の USB ポートが足りない
   - Webカメラも接続しているので一つ足らない
1. MX Keys と Thinkpad を Bluetooth で接続しないといけない
   - マウス・キーボードの Bluetooth 接続は避けたい

これを解決するために、マルチデバイス対応の Unifying 接続できる高精細マウスを探していた。
[Logicool MX Master](https://www.logicool.co.jp/ja-jp/products/mice/mx-master-3.910-005707.html)
が有力な候補だったが、サイズが少し大きい。

そこに MX Anywhere3 が発表された。

## 改善された環境

| 環境 | MX Anywhere3 (mouse) & MX Keys (keyboard) |
| -- | -- |
| Mac mini | USB レシーバ (Unifying) |
| Thinkpad | USB レシーバ (Unifying) |

MX Anywhere3 にも Unifying レシーバが同梱されているため、MX Keys のものと合わせて、
それぞれを Mac mini と Thinkpad 専用として使う形にしている。
Unifying レシーバは、[それ一つで最大6台のデバイスをペアリングできる](https://www.logicool.co.jp/ja-jp/promotions/6072) ため、
[Unifying Software](https://support.logi.com/hc/ja/articles/360025297913) を使って、それぞれに MX Keys と MX Anywhere3 を追加でペアリングすることができる。

なお、Unifying Software は Linux には提供されていない。自分は macOS でペアリングを設定したが、[Solaar](https://github.com/pwr-Solaar/Solaar) を使って Linux 上でペアリングしても良いと思う。

これによって、次のように劇的に環境が改善した。

- 環境を切り替える場合は、MX Anywhere3・MX Keys どちらも、接続先をボタンで切り替えるだけ
- それぞれ専有する USB ポートが一つで済む

## MX Anywhere 3 の感想

さて、前置きが長くなったが、MX Anywhere 3 を使ってみた感想を書いておく。

### :+1: Pros

- 十分に高精細で macOS/Ubuntu ともに思った通りに動かせてストレスがない
  - Ubuntu に関しては G304 は少し不満があったが、MX Anywhere 3 ではそれがない。これは予想外だった
- MagSpeed電磁気スクロールは、想像以上に便利
  - マウスホイールでごりごりスクロールする機会はあまりないが、4Kディスプレイの広作業スペースでは便利な機能かも (4Kディスプレイは、32インチ ViewSonic を使ってる)

### :-1: Cons

- 少し小さい。事前に大きさを確認したつもりだったが、もう一回り大きいとベストだった
- 電磁気スクロールは確かに静音だが、スクロールを止めるときにカチッという音がして気になる
- サイドボタンが少し固い

いくつか気になる点はあるが、基本的には満足している。

## Piper & libratbag

MX Anywhere3 のカスタマイズは、macOS であれば [Logicool Options](https://support.logi.com/hc/ja/articles/360025297893)
でできるが、Linux ではこれは使えない。しかし、有志で [Piper](https://github.com/libratbag/piper) というツールが開発されており、
これを使うことで、MX Anywhere 3 などの Logicool/Logitech 製品だけでなく、さまざまなデバイスのカスタマイズを GUI で行うことができる。感謝しかない。

なお、Piper は単なる GUI であり、デバイスの情報は [libratbag](https://github.com/libratbag/libratbag) で管理されている。
Piper がサポートしているデバイスを知りたい場合は、[libratbag/data/devices/](https://github.com/libratbag/libratbag/tree/master/data/devices) を参照すると良い。

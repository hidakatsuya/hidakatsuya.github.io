---
layout: post
title: Ubuntu 20.04 にアップグレードした
---

先日、Ubuntu の LTS 版である 20.04 がリリースされた。

![2020-05-13-ubuntu20 04-announcement](https://user-images.githubusercontent.com/739339/147848239-bfa402b5-2b88-4592-8766-efea24cad0a2.png)

https://ubuntu.com/blog/ubuntu-20-04-lts-arrives

現在のメインマシンは Ubuntu 19.10 だったので、早々にアップグレードしたかったのだが降って来ない。
GW中、時間を持て余していたということもあり、待ちきれず [公式ドキュメント](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes) に従って強制的にアップグレードした。

また、良い機会だったので、アップグレードの前に dotfiles 化も行い、最低限のバックアップを実施。

アップグレード自体は、特に問題なく...
とは行かず、途中「アップグレードに失敗したから元に戻すね」というメッセージに絶望するも、なぜかアップグレードは成功しているという、やや不安の残る結果に。

その後、アップグレードで無効化された apt source repository の整理など、いくつかの設定の調整を行ったが、その後は開発環境も含め問題なく動いている。あのメッセージはなんだったのか...

20.04 は派手な変更点や新機能はなく、セキュリティと安定性向上を目的としたバージョン ([ReleaseNote](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes/Ja)) のようだが、確かに 19.10 よりも安定していると思うし、非常に気に入っている。
また、20.04 には [Ubuntu 8.04 LTS の壁紙が収録](https://gihyo.jp/admin/serial/01/ubuntu-recipe/0616?page=2) されており、これがとても良い。

Ubuntu に移行してから半年以上が経過し、開発環境も整いつつあるので、現在の環境を別途まとめようと思う。

---
title: 猫のトイレ掃除当番を教えてくれる LINE Bot を Cloudflare Workers で動かす
---

## 猫のトイレ掃除当番の導入

我が家の猫のトイレ掃除を当番制にすることになったので、当番をうまく回すために仕組みを導入することにした。

* 当番は一日としローテーションする
* 毎日7時と18時に当日の当番を通知する
* 通知は家族の LINE グループに流す
* 当番の内容、今日/明日の当番を知ることができる
* 予算はゼロ

## LINE Touban Bot

猫のトイレ掃除当番を教えてくれる LINE bot（当番bot）を作り、家族の LINE グループに住まわせることにした。

当番bot は、毎日7時と18時にその日の当番をつぶやく。
![line1](https://github.com/hidakatsuya/hidakatsuya.github.io/assets/739339/2d3d0775-46e0-4e92-a0f4-f230ebb747a7)

当番bot に「今日/明日の当番は？」「当番？」と、問い合わせることができる。
![line2](https://github.com/hidakatsuya/hidakatsuya.github.io/assets/739339/b34a4ff2-3c5d-49cd-9658-0b8a65930b0e)

## アーキテクチャ

![image](https://github.com/hidakatsuya/hidakatsuya.github.io/assets/739339/29a024fb-9042-46bb-a507-5c8cf88e4a1f)

- Messaging API を有効にしたチャネルを作成して [LINE ボット](https://developers.line.biz/ja/docs/messaging-api/building-bot/#add-your-line-official-account-as-friend) を作成  
- Cloudflare Workers によって Webhook の受信先とスケジュールトリガーを構築
- 当番bot への問い合わせ時は LINE Platform から Cloudflare Workers の [Webhook を叩く](https://developers.line.biz/ja/docs/messaging-api/receiving-messages/)
- Webhook 受信時やスケジュールトリガー時に LINE グループへメッセージを送信するときは [Messaging API](https://developers.line.biz/ja/docs/messaging-api/getting-started/) を使う

## 実装

以下の GitHub レポジトリを参照

https://github.com/hidakatsuya/line-touban-bot

- Cloudflare Workers (wrangler-sdk)
- TypeScript
- Vitest (test)
- Biome (linter)

## 実装時につまづいた点、気になった点

- [LINE Messaing API SDK](https://github.com/line/line-bot-sdk-nodejs) は、v8.4.0 時点で Cloudflare Workers では使えない
  - "stream" や "querystring" などの Node.js パッケージのロードで失敗することが原因
  - Cloudflare Workers のランタイムは Node.js ではなく、[いくつかの Node.js API を互換サポートしている](https://developers.cloudflare.com/workers/runtime-apis/nodejs/)。また、Node.js API を import するときは "node:" プレフィックスが必要。しかし、LINE SDK では "node:" プレフィックスを指定していないため、インポートできない
- メッセージにメンションを含めることができない（たぶん）
  - [ドキュメント](https://developers.line.biz/ja/reference/messaging-api/#send-push-message) を読んでも対応していないように見える
- 当初は GAS を使おうとしたが、何らかの技術的な理由によって断念した
  - 理由は忘れた...

## まとめ

- 今の所いい感じに動いている。業務も改善した
- 薄々気づいていたけど、今日/明日の当番を問い合わせる機能は不要だったかも。誰も使ってない...
- Cloudflare Workers は本当に便利で簡単。もっといろいろ使いたい
- テストコードはほとんどが GitHub Copilot に書いてもらったコードを使っているので、一部冗長なコードがあるがこれで十分
- 後から知ったが、テストや開発向けに Cloudflare Workers をエミュレートすることができる [Miniflare](https://miniflare.dev/) というものがあるらしい。この bot を作っているときは、テスト用のグループを用意して実際に動かしながら行ったけど、これを使えばその辺りの手間を減らすことができそう。元気があれば、これを使ったテストコードに書き換えようかな

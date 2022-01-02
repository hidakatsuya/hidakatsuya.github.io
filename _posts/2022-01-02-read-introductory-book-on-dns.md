---
title: DNSの入門書を読み終えた
---

2021年4月ごろ、DNSを雰囲気でしか理解していないことを再認識する機会があった。
特段DNSに興味があるわけでもなく、自身の仕事の領域でも必須の知識というほどでもないため、まずは、読みやすさを重視した入門書を手に取ることにした。

## 書籍

<a href="https://hb.afl.rakuten.co.jp/ichiba/2304902d.f05c26bc.2304902e.7c640b7c/?pc=https%3A%2F%2Fitem.rakuten.co.jp%2Fbook%2F15653039%2F&link_type=text&ut=eyJwYWdlIjoiaXRlbSIsInR5cGUiOiJ0ZXh0Iiwic2l6ZSI6IjI0MHgyNDAiLCJuYW0iOjEsIm5hbXAiOiJyaWdodCIsImNvbSI6MSwiY29tcCI6ImRvd24iLCJwcmljZSI6MSwiYm9yIjowLCJjb2wiOjEsImJidG4iOjEsInByb2QiOjAsImFtcCI6ZmFsc2V9" target="_blank" rel="nofollow sponsored noopener" style="word-wrap:break-word;"  >DNSがよくわかる教科書 [ 株式会社日本レジストリサービス（JPRS） 渡邉結衣、 佐藤新太、 藤原和典 ]</a>

割とボリュームはあるが、文章は読みやすく、終始ストレスなく読めたと思う。内容は、DNSの背景や歴史から、ドメインとの関係や管理体制や DNS の名前解決の仕組みや仕様、DNSサービスの構築から `dig` コマンドによる動作確認とセキュリティ対策まで、
入門書として必要な内容は一通り抑えられていると思う。

全体の構成としては、前半で歴史や背景を説明し、中盤でDNSの仕様を説明。終盤で運用にフォーカス、といった形になっている。今回は、仕様を知ることが目的だったため、特に中盤を丁寧に読んだ。

## 印象に残った内容

覚えている範囲で、個人的に印象に残っているのが次の内容。

- DNSの名前解決における構成要素には「スタブリゾルバー」「フルリゾルバー」「権威サーバー」と呼ばれる3つの役割がある
  - これらの呼称は RFC 8499 で定められている
  - スタブリゾルバーは DNS クライアント、フルリゾルバーはキャッシュDNSサーバー、権威サーバーはゾーン・ネームサーバなどを指す
- 目的のリソースレコードが存在しない場合もキャッシュされ、これを「ネガティブキャッシュ」と呼ぶ
- 絶対ドメイン名は TLD まで省略なく表記し末尾にルートを表す `.` を付ける
  - e.g. `www.example.com.`
- ゾーンそのものの情報を SOA (Start of Authority) レコードに、委任に関する情報を NS レコードに設定する
- AAAA (クアッドA) リソースレコードは IPv6 のフォーマットである
- CNAME (Canonical Name) リソースレコードはドメイン名の正式名を指定するためのリソースレコードである
  - e.g. `www.hidakatsuya.dev.	3600	IN	CNAME	hidakatsuya.github.io.`
- DNSの動作確認を `dig` コマンドで行う場合、問い合わせ先がフルリゾルバーか権威サーバーかによって名前解決要求の有無を適切に変える必要がある
  - 問い合わせ先が権威サーバーの場合は、`+norecurse` を付与し名前解決を無効にし、フルリゾルバーの場合は逆にこれは付与しない
  

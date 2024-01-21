---
title: hidakatsuya.dev ドメインを Google Domains から Cloudflare へ移管した
---

hidakatsuya.dev を Google Domains から Cloudflare へ移管した。

## Google Domains から Cloudflare へ

すでに、手順を説明する記事は多くある。それらに従うことで、基本的には問題なく移管することができた。

その中でも、Cloudflare が提供している [ドメインをCloudflareに移管するためのステップバイステップガイド](https://blog.cloudflare.com/ja-jp/a-step-by-step-guide-to-transferring-domains-to-cloudflare-ja-jp/)
は分かりやすく、移管する際の全体像の把握のためにも非常に役に立った。

## ERR_TOO_MANY_REDIRECTS

移管完了後、 https://www.hidakatsuya.dev へアクセスすると、 `ERR_TOO_MANY_REDIRECTS` が出力されてページを開けない。

どうやら、Cloudflare 上の SSL/TLS の設定に問題があったと思われる。

https://zenn.dev/st_little/articles/fix-err-too-many-redirects-in-github-pages

---
title: www サブドメインを設定した
date: 2021-12-26 23:33:00 +0900
---

Apexドメインに加えて、www サブドメインにもサイトをホストするようにした。積みタスクの年末大掃除の一貫。

手順については、GitHub のドキュメントの
[Configuring an apex domain and the www subdomain variant](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain-and-the-www-subdomain-variant)
の通り行った。

```
$ dig www.hidakatsuya.dev +nostats +nocomments +nocmd
;www.hidakatsuya.dev.		IN	A
www.hidakatsuya.dev.	3600	IN	CNAME	hidakatsuya.github.io.
...
```

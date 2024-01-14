---
title: Kamal を使って RedMica を EC2 にデプロイする
---

Kamal を試した。

## Kamal

https://kamal-deploy.org/

Basecamp 社が開発した Docker コンテナ版 [Capistrano](https://capistranorb.com/) と言ったところ。

## RedMica

ちょうど [RedMica の公式 Docker イメージ](https://hub.docker.com/r/redmica/redmica/) があることを知ったので、RedMica をデプロイすることにする。

[RedMica](https://www.redmica.jp/) は Redmine 互換の OSS で Rails で作られている。

## EC2 にデプロイする

デプロイ先の環境は非常にシンプル。

![aws](https://github.com/hidakatsuya/hidakatsuya.github.io/assets/739339/8ec36760-6ea4-4b21-85e4-e2d0f7e52574)

基本的に、 [公式ドキュメント](https://kamal-deploy.org/docs/installation) に従って進めるだけでデプロイすることができた。

試した際のコードは、[hidakatsuya/redmica-deployment-with-kamal](https://github.com/hidakatsuya/redmica-deployment-with-kamal) に置いてある。

## 躓いたところ

とはいえ、いくつか躓いたところもあった。

### healthcheck

Kamal は、デフォルトでは `/up` でヘルスチェックを行う。しかし、RedMica v2.4.0 時点では `/up` は存在しない (Rails 6 ベースなので）。
今回は `/login` をヘルスチェック先として設定した。

```yaml
healthcheck:
  path: /login
  port: 3000
```

### Debian ベースのディストリビューション

Kamal は、Docker や curl などの必要なコマンドを自動的にインストールする機能を持っている。
この機能を使うためには、Debian ベースの OS である必要がある。`apt-get` でインストールするため。

なお、予めデプロイ先にこれらをインストールするのであれば、この制約の影響は受けない。

## まとめ

とても便利。GitHub Actions 等から実行することも想定されている。
しかし、実際にプロダクション環境で使う場合は、Kamal のアーキテクチャをしっかり理解する必要があると感じた（まだよくわかってない）。

---
title: GitHub Action でテスト用の Redmine/RedMica 環境をセットアップする action-setup-redmine を作った
---

Redmine プラグインのテストを実行するためには、Redmine 本体を準備する必要がある。準備自体は難しいことはないが面倒ではある。

というわけで、テスト用の Redmine 環境をセットアップする GitHub Action を作った。

https://github.com/hidakatsuya/action-setup-redmine

使い方は README の通り。

```yaml
- uses: hidakatsuya/action-setup-redmine@v1
  with:
    ruby-version: '3.2'

- uses: action/checkout@v4
  with:
    path: plugins/redmine_your_plugin

- run: |
    bundle install
    bin/rails redmine:plugins:test NAME=redmine_your_plugin
```

これだけで、[redmine/redmine](https://github.com/redmine) の master の Redmine でテストを実行することができる。

`with` にて、いくつかの設定が可能。

* `repository`: Redmine のソースリポジトリ。例えば、ディストリビューションの [RedMica](https://github.com/redmica/redmica) の場合は `redmica/redmica` とする。デフォルトは `redmine/redmine`
* `version`: リポジトリのタグやブランチ、コミットを指定。デフォルトは `master`
* `database`: Redmine で使うデータベース。SQLite3 (`sqlite3`) と PostgreSQL, MySQL (Docker公式のイメージタグ）を設定できる。デフォルトは SQLite3
* `ruby-version`: Ruby のバージョン。`ruby/setup-ruby` の `ruby-version` と同じ値を設定できる
* `path`: Redmine をセットアップするディレクトリ。例えば `redmine-src` を設定すると、`redmine-src` ディレクトリに Redmine がインストールされる。デフォルトはカレントディレクトリ

例えば、RedMica の v2.4 系の安定版の最新を PostgreSQL でセットアップしたい場合は次のようにする。

```yaml
- uses: hidakatsuya/action-setup-redmine@v1
  with:
    repository: 'redmica/redmica'
    version: 'stable-2.4'
    database: 'postgres:14'
    ruby-version: '3.2'
```

以上、詳細は README を参照。

元々、[redmine_ip_filter](https://github.com/redmica/redmine_ip_filter) のテスト環境を GitHub Action で作って pull request を投げようとしていて、
一通り書いたところで、Redmine のセットアップのステップを [composite アクション](https://docs.github.com/ja/actions/creating-actions/creating-a-composite-action) に切り出してリファクタリングし、
最終的に、その切り出したアクションを単独のアクションにして今に至る。

GitHub Action は本当によくできていて、こういうことが数時間もあればできてしまう。素晴らしい。

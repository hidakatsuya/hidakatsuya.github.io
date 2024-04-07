---
title: Redmine を開発するための Docked コマンド「redmined」
---

[Redmine](https://www.redmine.org/projects/redmine) の開発を簡単に始めることができる [redmined](https://github.com/hidakatsuya/redmined) という CLI を作った。

## Redmined

https://github.com/hidakatsuya/redmined

これは Rails の [Docked Rails CLI](https://github.com/rails/docked) に参考にしたもので、
開発に必要な環境を提供する Dockerイメージと、そのコンテナ上で `bin/rails server` などのコマンドを実行可能にする `redmined` コマンドを提供する。

Redmine はその Dockerコンテナ上で実行されるため、Docker と Redmine のソースコードがあればすぐに開発を始めることができる。もちろん、テストも実行できる。

## モチベーション

Redmine の完全なテストを実行するためには、いくつかの環境設定が必要で、それらを端末ごとにセットアップするのは面倒。
他の有力な選択肢として、VSCode の devContainer があるが、プライベートでは VSCode を使っていないこともあり、環境に依存しないことが望ましい。

## 使い方

インストール手順は [README](https://github.com/hidakatsuya/redmined?tab=readme-ov-file#installation) の通り。

あとは、Redmine のソースコードを公式ミラー [redmine/redmine](https://github.com/redmine/redmine) などから取得し、以下のコマンドを実行すればテスト可能な Redmine 環境ができあがる。

Redmine のディレクトリに移動して、データベース（SQLite）の設定を追加し、
```shell
cd your-redmine-root

cat <<EOS > config/database.yml
development:
  adapter: sqlite3
  database: db/development.sqlite3
test:
  adapter: sqlite3
  database: db/test.sqlite3
EOS
```
セットアップすれば起動できる。
```shell
redmined bundle install
redmined bin/rails db:prepare
redmined bin/rails s
```

コンソールに入りたい場合は、別のターミナルにて `redmined bin/rails c` などとすればいい。

## ファイルのオーナー問題

`bin/rails g scaffold blog` などのコマンドはコンテナ上で実行されるため、しばしばパーミッションエラーでファイルを編集できない問題が起きる（たぶん、Docker Desktop for mac では発生しない）。

`REDMINED_RUN_AS_HOST_USER` を有効にすることで、ホスト側のユーザーをコンテナユーザーにマッピングしてコンテナを実行することができる。

```bash
# .bashrc など
export REDMINED_RUN_AS_HOST_USER=1
```
もしくは、単に `redmined` 実行時にセットするだけでもいい。
```shell
REDMINED_RUN_AS_HOST_USER=1 redmined bin/rails s
```

## 制限

データベースは SQLite3 を前提にしていることと、LDAP のテストは実行できない。

このあたりを考え出すと、もう公式に開発用の compose.yml と Dockerfile を用意する方が良いように思う。

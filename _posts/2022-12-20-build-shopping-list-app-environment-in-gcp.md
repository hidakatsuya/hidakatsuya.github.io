---
title: Shopping List アプリの環境を Google Cloud Platform 上に構築して運用する
---

Heroku で運用していた Shopping List アプリを Google Cloud Platform に移行した。

Google Cloud Platform は何もわからない状態だったので、まずは公式ガイドの [Cloud Run 環境での Rails の実行](https://cloud.google.com/ruby/rails/run?hl=ja)
に従って動くものを構築し、それをベースに手を加えていく形で進めることにした。また、基本的に GCP の公式ガイドやサンプルに従って構築するようにした。

## Shopping List

個人用のお買い物リストアプリ。実際にほぼ毎日使っている。機能や構成は
[Rails と Turbo と Turbo Native for Android によるお買い物リストアプリを開発し運用する](2022-07-25-create-shopping-list-web-and-android-app-with-turbo.md)
の記事から基本的には変わっていない。

- [hidakatsuya/shopping_list](https://github.com/hidakatsuya/shopping_list)
  - Webアプリと REST API
  - Rails7 + Turbo + Propshaft + Tailwind CSS
- [hidakatsuya/shopping_list-android](https://github.com/hidakatsuya/shopping_list-android)
  - Android アプリ
  - Turbo Native for Android
- [hidakatsuya/shopping_list-cli](https://github.com/hidakatsuya/shopping_list-cli)
  - CLI, Go

## 方針

- 無料枠を最大限使い、可能な限り運用コストを0円にする
- 引き続きコードは public リポジトリに置く
- 環境構築に必要な設定は可能な限りコードで管理する
- Slack からデプロイできるようにする

## 全体の構成

[![gcp-shopping-list-structure](https://user-images.githubusercontent.com/739339/208452330-253e38f4-157e-435d-8b1a-063a400d596d.png)](https://user-images.githubusercontent.com/739339/208452330-253e38f4-157e-435d-8b1a-063a400d596d.png)

- Cloud SQL の無料枠はないので、Compute Engine (e2-micro) で PostgreSQL を動かす
  - 設定が面倒だったので、[こちら](https://joncloudgeek.com/blog/deploy-postgres-container-to-compute-engine/) を参考に PostgreSQL コンテナをデプロイして構築した
- Cloud Run から Compute Engine に接続するためには、VPC アクセスコネクタが必要
  - https://cloud.google.com/run/docs/configuring/connecting-vpc
- リージョンは us-west1 に統一
  - 無料枠を使えるリージョンが決まっている場合があるため

## Shopping List アプリの構成

- 機密情報の管理は `credentials.yml.enc` を使わない
  - 引き続きコードは public リポジトリに置いて開発する方針のため
- 必要な機密情報やパラメータはすべて Secret Manager で管理
- 本番イメージ用の Dockerfile を用意
  - https://github.com/hidakatsuya/shopping_list/blob/main/Dockerfile
  - 開発用のイメージとは異なる

## Cloud Build Executer

![Screenshot from 2022-12-20 01-40-03](https://user-images.githubusercontent.com/739339/208475597-bc72edb2-d6bb-4ea0-8ac7-c99253b0a709.png)

- Slack の Slash コマンドから Cloud Build を実行できる
  ```
  /cloud-build shopping-list
  ```
- 実行したい Cloud Build は Webhook トリガーを有効にする必要がある
- [Google Apps Script で公開した Web アプリ](https://gist.github.com/hidakatsuya/0be7f65816a6c09f4cc02c2c1108ebb6) に対して Slash コマンドから POST し、
Cloud Build の webhook トリガーを叩く
- 実際の Cloud Build の流れは [cloudbuild.yaml](https://github.com/hidakatsuya/shopping_list/blob/main/cloudbuild.yaml) を参照

## Cloud Build Notifier

![Screenshot from 2022-12-20 01-39-21](https://user-images.githubusercontent.com/739339/208477615-f2a16050-fff5-4197-9318-47f7b26127e5.png)

- Cloud Build の通知を Slack に流す
- Cloud Build のイベントは `cloud-builds` トピックに push される
- その `cloud-buids` トピックに向けた Pub/Sub をトリガーに Cloud Functions を実行し、受け取った通知を Slack Webhook 経由で Slack に流す
- Cloud Functions のコードは [hidakatsuya/gcp-cloud-build-slack-notifier](https://github.com/hidakatsuya/gcp-cloud-build-slack-notifier) においてある
- ちなみに、この Cloud Functions のデプロイも、前述の Slash コマンドから実行できる。こちらのデプロイ状況も Slack に通知される
  ```
  /cloud-build cloud-build-notifier` 
  ```
## まとめ

- 概ねいい感じに動いている
- 90日間のクレジットがある間に実際の費用を検証したいのだが、クレジットを適用しない場合の実際の費用を確認する方法がいまいちわからない。お支払い画面難しくない？
- IAM をみると「過剰な権限」の警告が大量に発生しているので整理する
- [公式ガイド](https://cloud.google.com/docs/terraform) を参考に Terraform で構成を管理しようとしている
- と、いろいろやりたいことは残っているが、ひとまず環境構築はこのぐらいにして Shopping List アプリの機能追加を進めたい

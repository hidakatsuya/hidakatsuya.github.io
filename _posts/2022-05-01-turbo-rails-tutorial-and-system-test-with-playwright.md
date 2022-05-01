---
title: The Turbo Rails チュートリアルのサンプルアプリケーションの system test を Playwright で動かすようにした
date: 2022-05-01 20:19:00 +0900
---

## The Turbo Rails Tutorial

少し前から [The Turbo Rails Tutorial](https://www.hotrails.dev/) に取り組んでいる。

このチュートリアルでは、[Quote editor](https://www.hotrails.dev/quotes) という Rails アプリケーショを作りながら
[turbo-rails](https://github.com/hotwired/turbo-rails) による SPA の開発について学ぶことができる。
全て英語だが、非常に読みやすいと思う。進め方や説明も丁寧でストレスがない。また、チュートリアルで作る Quote editor も完成度が高く実践的だと思う。
実際、チュートリアルを一通り終えたら、Quote editor を参考に個人的なアプリケーションを作る計画を立てている。

## Playwright

このチュートリアルでは、基本的に system test による TDD で開発を進めていく。
チュートリアルの system test は、selenium-webdriver を使うが、以前から気になっていた Playwright を使ってみることにした。
また、Chapter4 あたりで redis が必要になったこともあり、system test も含めて docker 環境の構築も行った。

[Playwright](https://github.com/microsoft/playwright) は、
> Playwright is a framework for Web Testing and Automation. It allows testing Chromium, Firefox and WebKit with a single API.

である。詳細は GitHub や [Getting Started](https://playwright.dev/docs/intro) にまとまっている。

さて、Playwright で system test を動かすと言っても、実のところは、Selenium に代えて Playwright の Capybara webdriver 実装である
[capybara-playwright-driver](https://github.com/YusukeIwaki/capybara-playwright-driver) を使うだけで済む。作者の方に感謝。

余談だが、この作者の方は Playwright の Ruby クライアント [playwright-ruby-client](https://github.com/YusukeIwaki/playwright-ruby-client) の作者でもある。
この作者の方の [紹介スライド](https://speakerdeck.com/yusukeiwaki/railsfalsesystem-speckara-playwrightwoshi-u) で Capybara から Playwright までわかりやすく説明されており、非常に勉強になった。

実装の対応内容は以下のコミットの通り。  

:octocat: [hidakatsuya/quote-editor:a2e6aa5 - system test works with playwright](https://github.com/hidakatsuya/quote-editor/commit/a2e6aa57421da35126d0b76556709caa60a72dc3)

driver を代えるだけ、と言ったが、実際にはもう少し複雑なので概要をまとめておく。

- [Playwright server/client 方式](https://playwright-ruby-client.vercel.app/docs/article/guides/playwright_on_alpine_linux#playwright-serverclient) を採用
- Playwright サーバー (server) は、公式の dockerイメージ [mcr.microsoft.com/playwright](https://hub.docker.com/_/microsoft-playwright) 使い playwright コンテナで起動する
- Capybara (client) は `Capybara.register_driver` で capybara-playwright-driver を登録し、Playwright サーバーのエンドポイント `ws://playwright:8888/ws` と対話する

以下の記事が参考になった。

- [Playwright on Alpin Linux](https://playwright-ruby-client.vercel.app/docs/article/guides/playwright_on_alpine_linux)
- [Capybara driver for Ruby on Rails application](https://playwright-ruby-client.vercel.app/docs/article/guides/rails_integration)
  - このページはあとから見つけたが Capybara のセットアップはこのページに全て書いてある
- [YusukeIwaki/capybara-playwright-driver](https://github.com/YusukeIwaki/capybara-playwright-driver)
- [Add options to execute Playwright.connect_to_playwright_server #44](https://github.com/YusukeIwaki/capybara-playwright-driver/pull/44)

今回雰囲気で使ってみたが、テスト対象のアプリケーションがシンプルなものとはいえ、system test はその後も安定して動作している。今後も積極的に使っていきたい。

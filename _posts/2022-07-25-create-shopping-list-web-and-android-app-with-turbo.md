---
title: Rails と Turbo と Turbo Native for Android によるお買い物リストアプリを開発し運用する
---

予てより Hotwire に興味があったこともあり、Hotwire と Rails でプライベートの実用的なアプリを作って運用することにした。

## お買い物リストアプリ

題材はお買い物アプリとした。単純だし、日頃からよく使う。

要件はこんなところ。

- 買い物アイテムの追加、編集、削除ができる
- 買い物アイテムを完了状態に更新できる
- Google アカウントでサインアップ、サインインできる
- Alexa や Google Home から音声でアイテムを追加できる（これが一番大事）
- モバイル(Android 12)でも快適に利用できる

## 方針

3つのプロダクトを作る。優先順。

1. Webアプリ
   - Rails7 + Turbo + Propshaft + Tailwind CSS
   - Heroku で運用
2. Androidアプリ
   - Turbo Native for Android
   - ダークモードに対応
3. CLI
  ```
  $ shopping-list add 牛乳
  ```

## Webアプリ

https://github.com/hidakatsuya/shopping_list

<div>
  <img src="https://user-images.githubusercontent.com/739339/180660206-cd47ac88-21aa-4fac-a085-8d2b4b888c50.png" width="30%">
  <img src="https://user-images.githubusercontent.com/739339/180660241-18e6755f-d20b-4e2f-a181-8c13fbeec0da.png" width="30%">
</div>

- Googleアカウントのサインイン・サインアップは Devise と OmniAuth Google OAuth2 Strategy で実装
  - [Devise の wiki](https://github.com/heartcombo/devise/wiki/OmniAuth%3A-Overview) が非常に参考になった
  - パスワード認証を無効にして、Googleアカウント認証のみとする場合の説明もこの wiki できちんと説明されている
- Bootstrap5 via CDN から TailwindCSS via tailwindcss-rails へ移行
  - ダークモードの対応のために tailwindcss-rails へ移行: [hidakatsuya/shopping_list#19](https://github.com/hidakatsuya/shopping_list/pull/19)
  - cssbunding-rails + Bootstrap というオプションもあったが、どうせなら node に依存しないままにしたかったため採用
  - tailwindcss-rails は `tailwindcss:build` を test なら`test:prepare`、production なら `assets:precompile` で実行してくれるので、導入によって CI やデプロイプロセスを変更する必要はない
- Materal Design 2 のデフォルトテーマの配色を再現
  - Turbo Native for Android にて Android アプリとして動作するときに違和感が無いようにするために Material Design 2 の配色を再現した
  - https://material.io/design/color/dark-theme.html#ui-application
- Tailwind とプレーンな CSS で CSS 変数を共有
  - Rails のバリデーションエラーのスタイルは Tailwind 外のプレーンな CSS で定義する必要があった: [該当コード](https://github.com/hidakatsuya/shopping_list/blob/bf629ee5ac9c5b87f831f3ec5bb9b607fc7ec1c4/app/assets/stylesheets/application.css#L2-L12)
  - 手順は https://tailwindcss.com/docs/customizing-colors#using-css-variables の通り
- CI の高速化
  - [GitHub Actions での Docker によるテストの高速化を試みた](2022-06-11-try-to-make-github-action-faster.md) を参照
- Google IdToken によるサインイン
  - WebView では Googleログインできないため、Android はネイティブの Googleログインによって認証を行い、取得した IdToken を `/mobile/sign_in?token=<token>` に投げる形でセッションを構築する
  - https://github.com/hidakatsuya/shopping_list/commit/522c4ea68b558b5186602a7952d27516ff07f951
- Alexa/Google Home 連携
  - スキルを作るのはハードルが高いので、IFTTT などで `POST /items` API を叩くことで簡単に連携できる
  - Google Home だと話した内容を簡単に取り出すことができる。[参考](https://dare-tame.com/gh-line-send/)

## Androidアプリ

https://github.com/hidakatsuya/shopping_list-android

<div>
  <img src="https://user-images.githubusercontent.com/739339/180681717-b1600ff7-60c1-413f-9fed-0e367f48ffb9.png" width="30%">
  <img src="https://user-images.githubusercontent.com/739339/180681757-71431e00-9c61-4f42-9149-1e8d7b8408d8.png" width="30%">
</div>

- 公式の [turbo-android](https://github.com/hotwired/turbo-android) の demo アプリと公式ドキュメントがベース
  - Android Studio にて、空の Android プロジェクトを作成し、turbo-android の demo アプリのコードを少しずつ移植しながら実装した
  - [公式ドキュメント](https://github.com/hotwired/turbo-android#documentation) に一通りの説明が記載されている
- Google ログインはネイティブ Google Sign-in と IdToken によって実現
  - WebView では Google ログインできないため、Webアプリの Googleログインは Androidアプリでは動かない
  - Androidアプリのサインインは、ネイティブの Google ログインによって別途実装した
  - ネイティブの Google ログインで認証を行い、取得した Google IdToken を Webアプリで検証してセッションを構築する形
  - [Turbo Native Google Sign In Demo](https://github.com/kzkn/turbo-native-google-signin-demo) が非常に参考になった。感謝
  - その他には以下を参考にした
    - [Start Integrating Google Sign-In into Your Android App ](https://developers.google.com/identity/sign-in/android/start-integrating)
    - [Authenticate with a backend server](https://developers.google.com/identity/sign-in/android/backend-auth)
    - [An ID token's payload](https://developers.google.com/identity/protocols/oauth2/openid-connect#an-id-tokens-payload)
- Client ID などの環境ごとの機密情報は `x.properties` ファイルにて管理
  - 環境ごと（ビルドバリアントごと）の機密情報は Secrets Gradle Plugin for Android の Build-Variant Specific Properties によって管理している
  - https://github.com/google/secrets-gradle-plugin#build-variant-specific-properties
  - `debug.properties` や `release.properties` に定義しておけばいい感じに build してくれる
- 実機では Google ログインでクラッシュ
  - エミュレーター上では debug/release のどちらのビルドバリアントでも動作は問題ないが、実機 (Xperia 10Ⅲ) に apk 形式で動かすと Google ログイン時にクラッシュする問題がある
  - 原因は不明。実機でデバッグして調査しようしているところ

## CLI

現在 go で作っているところだが、最初のバージョンでは`POST /items` API を使った非常にシンプルな CLI になる。

```
$ shopping-list add 牛乳
```

## 感想

単純なシステムだが、いざゼロから作ると非常にたくさんの気づきや学びがあってとても有意義だった（道半ばだが）。
Android アプリの開発は、以前 Flutter に挑戦して挫折した程度の知識だったため、[ライフサイクル](https://atmarkit.itmedia.co.jp/ait/articles/1604/04/news011_3.html) など、
基本的なアーキテクチャから勉強する良い機会にもなった。その際、公式のドキュメント [Android for developer](https://developer.android.com/) が非常に充実していて情報に困ることはなかった。

また、やはり技術を学ぶ最適な手段は、自分が実際に使うものを自分で調べて作って、そして運用することだと感じた。
買い物リストという単純なシステムではあるが、この開発と運用を通じて実にこれだけのことを新たに学ぶことができている。

- Rails7
- Turbo
- Tailwind CSS
- Propshaft
- Omniauth Google Auth
- System Test と Cuprite/Playwright
- Turbo Native for Android
- Android アーキテクチャとエコシステム
- Material Design
- Android Studio
- Go
- Google Sign-in
- Heroku

買い物リストアプリは、わざわざ作らなくても、すでに世の中には無料で使いやすいものが多く存在するが、
世の中に存在しないものを思いつくことは昨今では困難。題材となるアプリを考えるときは、その観点は捨て、自分が今使っているツールやアプリを自身で再発明するのが良いのではないかと思う。





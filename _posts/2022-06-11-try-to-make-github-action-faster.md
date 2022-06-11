---
title: GitHub Actions での Docker によるテストの高速化を試みた
date: 2022-06-11 17:15:00 +0900
---

現在個人的な Rails アプリを作っている。

- docker-compose で環境を構築
- GitHub Actions 上でのテストも docker-compose にて実行
- Rails7 + importmaps-rails + Propshaft
- System Test の driver は [Playwright driver for Capybara](https://github.com/YusukeIwaki/capybara-playwright-driver) を採用

このテストの高速化の試みを簡単にまとめる。

## 最初の状態

docker-compose.yml と .github/workflows/test.yml は次の通り。

いずれも高速化や最適化は考慮していない最もシンプルな実装だが、 毎回 docker build しなくて済むように、app, playwright サービスは
予めビルド済みのイメージをコンテナレジストリに保存してある。また、app イメージにはアプリ固有の情報を保持しないようにしてある。

{% raw %}

```yml
# docker-compose.yml
version: "3"

services:
  app:
    image: ghcr.io/hidakatsuya/rails-dev:ruby-3.1.0
    command: bash -c 'rm -f tmp/pids/server.pid; bin/rails s -b 0.0.0.0'
    tty: true
    stdin_open: true
    ports:
      - '3000:3000'
    volumes:
      - .:/app:cached
      - bundle:/bundle
    environment:
      PLAYWRIGHT_URL: 'ws://playwright:8888/ws'
    depends_on:
      - postgres

  postgres:
    image: postgres:14-alpine
    ports:
      - '15432:5432'
    volumes:
      - pg-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password

  playwright:
    image: ghcr.io/hidakatsuya/playwright-test-server:latest
    ports:
      - '8888:8888'
    command: --port 8888 --path /ws

volumes:
  bundle:
  pg-data:
```

{% endraw %}

{% raw %}

```yml
# .github/workflows/test.yml
name: Test

on: push

jobs:
  system_test:
    name: Test

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Build and setup
      run: |
        docker-compose pull
        docker-compose run app bin/setup
        docker-compose up -d

    - name: Test
      run: docker-compose exec -T app bin/rails test:all
```

{% endraw %}

このときの実行時間は 2m 前後。

![s1](https://user-images.githubusercontent.com/739339/173177018-fa6e7797-c15d-4b88-9ad6-7a626a45797e.png)

## Gem のキャッシュ

毎回 `docker-compose run app bin/setup` で `bundle install` しなくて済むように、`bundle install` の結果をキャッシュする。

{% raw %}

```diff
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -10,7 +10,7 @@ services:
       - '3000:3000'
     volumes:
       - .:/app:cached
-      - bundle:/bundle
+      - ${BUNDLE_STORE_PATH:-./tmp/bundle}:/bundle
     environment:
       PLAYWRIGHT_URL: 'ws://playwright:8888/ws'
     depends_on:
@@ -32,5 +32,4 @@ services:
     command: --port 8888 --path /ws

 volumes:
-  bundle:
   pg-data:
```

```diff
--- a/.github/workflows/test.yml
+++ b/.github/workflows/test.yml
@@ -2,6 +2,10 @@ name: Test

 on: push

+env:
+  BUNDLE_STORE_PATH: /tmp/bundle
+  RUBY_VERSION: '3.1.0'
+
 jobs:
   system_test:
     name: Test
@@ -9,12 +13,30 @@ jobs:
     runs-on: ubuntu-latest

     steps:
-    - uses: actions/checkout@v2
+    - uses: actions/checkout@v3
+    - uses: ruby/setup-ruby@v1
+      with:
+        ruby-version: ${{ env.RUBY_VERSION }}
+
+    - name: Cache
+      id: cache
+      uses: actions/cache@v3
+      with:
+        path: ${{ env.BUNDLE_STORE_PATH }}
+        key: ${{ env.RUBY_VERSION }}-gems-${{ hashFiles('Gemfile.lock') }}
+        restore-keys: |
+          ${{ env.RUBY_VERSION }}-gems-
+
+    - name: Bundle
+      if: steps.cache.outputs.cache-hit != 'true'
+      run: |
+        bundle config path $BUNDLE_STORE_PATH
+        bundle install --jobs 4 --retry 3

     - name: Build and setup
       run: |
         docker-compose pull
-        docker-compose run app bin/setup
+        docker-compose run app bin/rails db:prepare
         docker-compose up -d

     - name: Test
```

{% endraw %}

環境変数 `BUNDLE_STORE_PATH` で bundle ボリュームのホストパスを指定できるようにしている。
```diff
-      - bundle:/bundle
+      - ${BUNDLE_STORE_PATH:-./tmp/bundle}:/bundle
```

事前に、`BUNDLE_STORE_PATH` に設定したパスに対して bundle install を行い、`actions/cache` によってキャッシュする。
キャッシュがある場合は、bundle install はせず、`actions/cache` によって `BUNDLE_STORE_PATH` にリストアされる。
これによって`bin/setup` で bundle install を実行する必要がなくなったため、`bin/rails db:prepare` のみ明示的に実行するように変更。

キャッシュがある場合の実行時間は 1m30s ほど。30s 改善。

![image](https://user-images.githubusercontent.com/739339/173177740-cdd5d4e5-9fe0-4ab5-9b51-c8409e641414.png)

## Docker イメージのキャッシュ

Playwright のイメージは 2.3GB とサイズが大きいため、Docker イメージをキャッシュして docker pull をスキップして高速化する。

が、先に結論を書くと、こちらは高速化に至らなかった。

`.github/workflows/test.yml` は次の通り。差分が大きいため全体を示す。[Gist](https://gist.github.com/hidakatsuya/a20985499939d47ba23e08c0cdeacc62) にも置いてある。


{% raw %}

```yml
# .github/workflows/test.yml
name: Test

on: push

env:
  RUBY_VERSION: '3.1.0'
  PLAYWRIGHT_IMAGE_VERSION: latest
  CACHE_BUNDLE_PATH: /tmp/bundle
  CACHE_PLAYWRIGHT_PATH: /tmp/playwright-image

jobs:
  setup-gems:
    name: Setup gems

    runs-on: ubuntu-latest
    timeout-minutes: 3

    steps:
    - uses: actions/checkout@v3
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ env.RUBY_VERSION }}

    - name: Cache gem
      id: cache-gem
      uses: actions/cache@v3
      with:
        path: ${{ env.CACHE_BUNDLE_PATH }}
        key: gems-${{ env.RUBY_VERSION }}-${{ hashFiles('Gemfile.lock') }}
        restore-keys: |
          gems-${{ env.RUBY_VERSION }}-

    - name: Bundle
      if: steps.cache-gem.outputs.cache-hit != 'true'
      run: |
        bundle config path $CACHE_BUNDLE_PATH
        bundle install --jobs 4 --retry 3

  setup-playwright-image:
    name: Setup playwright image

    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
    - name: Cache image
      id: cache-image
      uses: actions/cache@v3
      with:
        path: ${{ env.CACHE_PLAYWRIGHT_PATH }}
        key: playwright-image-${{ env.PLAYWRIGHT_IMAGE_VERSION }}

    - name: Pull and save
      if: steps.cache-image.outputs.cache-hit != 'true'
      run: |
        docker pull ghcr.io/hidakatsuya/playwright-test-server:$PLAYWRIGHT_IMAGE_VERSION
        docker save ghcr.io/hidakatsuya/playwright-test-server:$PLAYWRIGHT_IMAGE_VERSION -o $CACHE_PLAYWRIGHT_PATH

  test:
    name: Test
    needs:
      - setup-gems
      - setup-playwright-image

    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
    - uses: actions/checkout@v3

    - name: Restore gem cache
      id: restore-gem-cache
      uses: actions/cache@v3
      with:
        path: ${{ env.CACHE_BUNDLE_PATH }}
        key: gems-${{ env.RUBY_VERSION }}-${{ hashFiles('Gemfile.lock') }}
        restore-keys: |
          gems-${{ env.RUBY_VERSION }}-

    - name: Ensure cache-hit for gem
      if: steps.restore-gem-cache.outputs.cache-hit != 'true'
      run: |
        echo "Gem cache not found"
        exit 1

    - name: Restore docker image cache
      id: restore-image-cache
      uses: actions/cache@v3
      with:
        path: ${{ env.CACHE_PLAYWRIGHT_PATH }}
        key: playwright-image-${{ env.PLAYWRIGHT_IMAGE_VERSION }}

    - name: Ensure cache-hit for playwright image
      if: steps.restore-image-cache.outputs.cache-hit != 'true'
      run: |
        echo "Playwright image cache not found"
        exit 1

    - name: Load playwright image
      run: docker load -i $CACHE_PLAYWRIGHT_PATH

    - name: Build and setup
      run: |
        docker-compose run app bin/rails db:prepare
        docker-compose up -d

    - name: Test
      run: docker-compose exec -T app bin/rails test:all
```

{% endraw %}

Docker イメージと gem の準備は、ジョブを並列化して時間を短縮している。ジョブの構成は次に通り。

![image](https://user-images.githubusercontent.com/739339/173178768-72f5f9d3-873c-4624-b157-834a1727e4cf.png)

gem のキャッシュの時と同様に `actions/cache` を使って docker pull/save にてイメージをキャッシュし、
docker load にてキャッシュを読み出すことで docker pull は不要になった。

gem と Docker イメージ両方のキャッシュがある状態での実行時間は 2m40s 前後。1m 程度遅くなってしまった。

![image](https://user-images.githubusercontent.com/739339/173179335-b5094b83-6959-4f50-b1b2-541cf83ba85e.png)

docker pull は必要なくなったが、代わりに、キャッシュからのリストア `Restore docker image cache` に 20s、
リストアした Docker イメージのロード `Load playwright image` に 40s 程度時間がかかるようになってしまった。

Docker 難しい。

## 参考

- [GitHub ActionsでDockerイメージをキャッシュする](https://dev.classmethod.jp/articles/github-actions-cache-docker-images/)



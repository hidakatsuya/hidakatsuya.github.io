---
title: thinreports.org のコンテンツを thinreports リポジトリの README.md に自動的に同期させるようにした
---

一週間ほど前に www.thinreports.org をリニューアルした。

## リニューアルの概要

- 今後 Thinreports の情報は [thinreports/thinreports](https://github.com/thinreports/thinreports) リポジトリに集める
- www.thinreports.org は、ドキュメントなどの必要なコンテンツを thinreports/thinreports に移行した上で全て削除する
- www.thinreports.org には index.html のみ配置し、[thinreports/thinreports の README.md](https://github.com/thinreports/thinreports/blob/main/README.md) と同期する

詳細は [Renewal #2](https://github.com/thinreports/thinreports.github.io/pull/2) を参照して欲しいが、最後の「README.md との同期」については少しまとめておくことにする。

## README.md と同期する

ポイントは次の二点。

- thinreports/thinreports の README.md の変更が www.thinreports.org の index.html に自動的に反映されるようにする
- www.thinreports.org のスタイルも合わせる（個人的な好み）

### スタイルの同期

スタイルの同期については、[pages-themes/primer](https://github.com/pages-themes/primer) の Jekyll テーマを使うことで簡単に解決できる。

### README.md の変更を自動的に反映させる

README.md と index.html の同期については、いくつかの選択肢を検討した上で [GitHub Action のスケジュールイベント](https://docs.github.com/ja/actions/learn-github-actions/events-that-trigger-workflows#schedule) を使って、定期的に index.html を更新する方法で解決することにした。

1. 毎日 0:00 (UTC) ごろに [ワークフロー](https://github.com/thinreports/thinreports.github.io/blob/master/.github/workflows/sync.yml) を実行
2. thinreports/thinreports の README.md の内容を取得
3. index.md の [Front Matter](http://jekyllrb-ja.github.io/docs/front-matter/)（先頭の `---` で囲まれた内容）と、取得した README.md の内容で index.md を更新する
4. index.md に差分があれば push する。差分がなければ何もしない

他の選択肢の検討など、詳細は [Sync index.md with README.md in thinreports/thinreports #4](https://github.com/thinreports/thinreports.github.io/pull/4) を参照して欲しい。

現在、新しいテンプレート形式である [Section Format の開発](https://github.com/orgs/thinreports/projects/1) を少しづつだが進めている。それに伴い、しばらくの間は Thinreports の情報更新が頻繁に発生することになるが、その場合でも [tinreports/thinreports](https://github.com/thinreports/thinreports) リポジトリのみ更新すればよく、www.thinreports.org との情報の齟齬も生じなくなった。更新の作業量も減少し、管理するリポジトリも大幅に削減できた。



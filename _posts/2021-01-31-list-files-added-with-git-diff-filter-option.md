---
layout: post
title: git の --diff-filter オプションを使って、追加されたファイルを列挙する
---

例えば、 [このサイトのリポジトリ](https://github.com/hidakatsuya/hidakatsuya.github.io)
のコミット `f055db4` から `5221edb` の中で、「追加されたファイルのパス」を列挙する場合は次のようにする。

```
$ git diff --name-only --diff-filter=A f055db4 5221edb
_posts/2020-12-13-got-playstation5.md
_posts/2021-01-01-prawn-emoji-v4.2.0-released.md
_posts/2021-01-18-thinreports-0.12.0-released.md
```

また、変更ステータスを列挙する場合は `--name-status` を使う。

```
$ git diff --name-status f055db4 5221edb
M       Gemfile.lock
A       _posts/2020-12-13-got-playstation5.md
A       _posts/2021-01-01-prawn-emoji-v4.2.0-released.md
A       _posts/2021-01-18-thinreports-0.12.0-released.md
M       about-me.md
```

git は長らく使ってるが、まだまだ未知の便利機能がありそう。

- [--diff-filter](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203)
- [--name-status](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---name-status)

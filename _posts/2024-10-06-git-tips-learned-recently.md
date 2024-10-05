---
title: 最近知った Git の使い方
---

最近知って衝撃だった Git の使い方をまとめる。今回の件で「面倒だなと思ったらまず調べよう」という教訓を得た（ここに挙げたものは、いずれも面倒だなとずっと思っていたものばかり）。

## git push orign HEAD

https://git-scm.com/docs/git-push より

> git push origin HEAD
> A handy way to push the current branch to the same name on the remote.

例えば、`new-feature` というローカルブランチで作業していて、その内容を同名のリモートブランチに push するとき、これまでは次のようにしていた。
```
git push origin new-feature
```

前述のドキュメントにはっきり書かれている通り、そんなことをしなくても常にこれでいい。
```
git push origin HEAD
```

## git branch --show-current

https://git-scm.com/docs/git-branch より

> --show-current
> Print the name of the current branch. In detached HEAD state, nothing is printed.

現在作業しているローカルブランチ名「だけ」を出力する。ただそれだけ。

だけど、Git 2.22 でこのオプションが追加されるまでは、次のような直感的でない方法しかなかった。
```
git rev-parse --abbrev-ref HEAD
```

StackOverflow の [この辺](https://stackoverflow.com/questions/6245570/how-do-i-get-the-current-branch-name-in-git) でさまざまな直感的でない方法が紹介されている。

Git 2.22 以降の現代は `git branch --show-current` でいい。

## GIT_AUTHOR_NAME, GIT_AUTHOR_EMAIL, GIT_AUTHOR_EMAIL, GIT_COMMITTER_EMAIL

https://git-scm.com/docs/git-config#Documentation/git-config.txt-username より

> user.name  
> user.email  
> author.name  
> author.email  
> committer.name  
> committer.email  
>   The user.name and user.email variables determine what ends up in the author and committer fields of commit objects. If you need the author or committer to be different, the author.name, author.email, committer.name, or committer.email variables can be set. All of these can be overridden by the GIT_AUTHOR_NAME, GIT_AUTHOR_EMAIL, GIT_COMMITTER_NAME, GIT_COMMITTER_EMAIL, and EMAIL environment variables.

上記説明の通り。`git config --global user.name hidakatsuya` や `user.email` を設定する代わりに、これらの環境変数にセットしておくこともできる。

## Git のリリースノート

Git のリリースノートの元データは同リポジトリの [Documentation/RelNotes](https://github.com/git/git/commits/master/Documentation/RelNotes) にある。
しかし、この中から今回取り上げたような機能を知ることは現実的ではない。

現状では、GitHub Blog に投稿されるまとめ [Highlights from Git 2.46](https://github.blog/open-source/git/highlights-from-git-2-46/) の記事を読むのが良さそう。
これらの記事は、 [Home/Open Source/Git](https://github.blog/open-source/git/) や [Hightlight from Git の検索結果](https://github.blog/?s=Highlights+from+Git) で一覧できる。

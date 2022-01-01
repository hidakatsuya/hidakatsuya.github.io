---
layout: post
title: GitHub CLI と fzf で目的の pull request を素早くチェックアウトする
---

`gh` を使うと pull request の ID から簡単にチェックアウトできる。

```
$ gh pr checkout 1234
```

これも十分便利だが、[Scripting with GitHub CLI - The GitHub Blog](https://github.blog/2021-03-11-scripting-with-github-cli/) で紹介されている 
[fzf と組み合わせた方法](https://github.blog/2021-03-11-scripting-with-github-cli/#combine-gh-with-other-tools) が最高だったので早速導入した。

例えば、[rails/rails](https://github.com/rails/rails/pulls) の pull request から目的のものをチェックアウトする場合はこんな感じだ。

![2021-04-10-checkout-pr-using-gh-with-fzf](https://user-images.githubusercontent.com/739339/147848214-c471fcd6-5067-4c34-a299-df9167b1ff1a.gif)

`gh` はリリース当初から使っているが、改めて調べてみると面白い。
特に [Using gh in GitHub Actions](https://github.blog/2021-03-11-scripting-with-github-cli/#using-gh-in-github-actions) にある通り、
GitHub Actions と組み合わせていろいろ遊んでみようと思う。

---
layout: post
title: 複数環境の dotfile を扱えるようにした
---

現在、複数の作業環境がある。プライベートは ubuntu、仕事は macOS だ。
元々、ubuntu 用に簡易的な dotfile 環境を運用していたが、macOS にも導入したかったため、今回複数環境の dotfile 環境を構築した。

自分の dotfile 環境は private リポジトリにしているので、Gist に内容をまとめた。詳細はこちらをみて欲しい。

[Gist:Simple and Cusomizable Dotfile Management](https://gist.github.com/hidakatsuya/6e22264049eb60c23749d5691225a50d)

特徴は次の通り。

- インストールなどのコマンドは `Rakefile` で実装
- 環境ごとに yml を用意し、dotfile とそのリンク先を定義する

例えば、`dotfiles` ディレクトリに dotfile を好きなように配置し、以下のような yml を用意する。

```yml
# macOS.yml
common:
  git:
    ignore: .config/git
  rubygems:
    .gemrc: .
  vim:
    .vimrc: .

macOS:
  bash:
    .bash_profile: .
    .bashrc: .
```

`common: git: ignore` は `$HOME/dotfiles/common/git/ignore` を指し、その値 `.config/git` は `$HOME/.config/git` ディレクトリを指している。
つまり、 `$HOME/dotfiles/common/git/ignore` は `$HOME/.config/git/ignore` にリンクすることを定義している。

また、この yml を定義すると `rake install:macOS` コマンドが利用できるようになるので、あとはこれを実行すればリンクが作成される。

```
$ rake install:macOS
[already_linked] .config/git/ignore
[already_linked] .gemrc
[already_linked] .vimrc
[link_created] .bash_profile (backup)
[link_created] .bashrc (backup)
[link_created] .gitconfig (backup)
```

`[already_linked]` はすでにリンクが存在するため処理がスキップされたことを意味し、`[link_created]` はリンクが作成されことを意味する。
また、`(backup)` は、リンクパスにすでにファイルが存在していたため、そのファイルが `$HOME/dotfiles/backup/YYYYMMDDHHIISS/filename` にバックアップされたことを意味している。

割と良い環境になったと思う。Ruby 使いやすい。

(追記) この dotfile 環境を構築するためのツールを [flexdot という名前の gem にして公開した](https://hidakatsuya.github.io/2020/07/21/flexdot-1.0.0-released.html)。

---
layout: post
title: diff-pdf をインストールする GitHub Action setup-diff-pdf をリリースした
---

[diff-pdf](https://github.com/vslavik/diff-pdf) をインストールする GitHub Action [hidakatsuya/setup-diff-pdf](https://github.com/hidakatsuya/setup-diff-pdf) をリリースした。

![](/images/2021-04-17-setup-diff-pdf.png)

diff-pdf は、名前の通り、2つの PDF ファイルの差分のチェックや、差分のビジュアライズ結果を生成できるツールである。
特に、PDF を扱うプロダクトのビジュアルテストツールとして非常に有用だ。

自身のプロダクトでも利用させてもらっている。

- [thinreports-generator](https://github.com/thinreports/thinreports-generator) - Report Generator for Ruby
- [prawn-emoji](https://github.com/hidakatsuya/prawn-emoji) - An extention that adds Emoji support to Prawn

しかし、Ubuntu ではソースからビルドしなければならないため、GitHub Action の Ubuntu で使うためにはひと手間必要になる。

```yaml
- name: Set up diff-pdf
  run: |
    sudo apt update
    sudo apt install make automake g++ libpoppler-glib-dev poppler-utils libwxgtk3.0-gtk3-dev
    git clone https://github.com/vslavik/diff-pdf.git -b v0.5 --depth 1 /tmp/diff-pdf-src
    cd /tmp/diff-pdf-src
    ./bootstrap && ./configure && make && sudo make install
```

[hidakatsuya/setup-diff-pdf](https://github.com/marketplace/actions/setup-diff-pdf) ならこれだけ。

```yaml
- uses: hidakatsuya/setup-diff-pdf@v1
  with:
    diff-pdf-version: 0.5
```

現在の v1.0.0 では `ubuntu-latest` と `windows-latest` をサポートしている。


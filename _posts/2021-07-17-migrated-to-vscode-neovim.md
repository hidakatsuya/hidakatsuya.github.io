---
layout: post
title: vscode-neovim に移行した
---

[VSCodeVim](https://github.com/VSCodeVim/Vim) から [vscode-neovim](https://github.com/asvetliakov/vscode-neovim) に移行した。

`init.vim` は次の通り。vim プラグインの管理は [vim-plug](https://github.com/junegunn/vim-plug) を使っている。

```vim
call plug#begin(has('nvim') ? stdpath('data') . '/plugged' : '~/.vim/plugged')

Plug 'tpope/vim-surround'
Plug 'preservim/nerdcommenter'

call plug#end()

set clipboard+=unnamedplus

" <Leader>
let mapleader = "\<Space>"
```

VSCodeVim も素晴らしい extension で何か問題があったわけではない。

強いて言えば、(少なくとも自分の環境では) 入力した絵文字を正しく扱えない場合があり、[prawn-emoji](https://github.com/hidakatsuya/prawn-emoji) などの絵文字を扱うコードを書く場合に少し不便ではあった。
詳しく調べていないが、https://github.com/VSCodeVim/Vim/issues/2722 が類似する。

---
title: OpenAI の API key などの機密情報を Passwords and Keys に保存して扱う
---

## APIキーなどの機密情報をどこに保存するか

OpenAI の APIキーは [公式ドキュメント](https://platform.openai.com/docs/quickstart?context=node) に従い、`~/.bashrc` に環境変数として定義していた。

```bash
export OPENAI_API_KEY='your-api-key-here'
```

しかし、`~/.bashrc` は dotfiles の管理化にあるため、API キーを含めることはできない。

なお、OS は Ubuntu 22.04 LTS でシェルは bash を使っている。

## Passwords and Keys (Seahorse) に保存する

Ubuntu に標準で付属している Passwords and Keys に保存し、`secret-tool` コマンドでロードする。

`secret-tool` で読み出し、環境変数に設定する。
```bash
# .bashrc
export OPENAI_API_KEY='$(secret-tool lookup service openai.com/chatgpt)'
```

なお、`secret-tool` で保存することもできる。
```shell
$ secret-tool store --label='OpenAI API key' service openai.com/chatgpt
```

## secret-tool

`secret-tool` の使い方はパラメータ無し（正しくないパラメータ）で実行すると出力される。

```shell
$ secret-tool
usage: secret-tool store --label='label' attribute value ...
       secret-tool lookup attribute value ...
       secret-tool clear attribute value ...
       secret-tool search [--all] [--unlock] attribute value ...
       secret-tool lock --collection='collection'
```

`attribute value` はドキュメントにある通り、キーを特定するための一意な値にしておく必要がある。

> Items are uniquely identified by a set of attribute keys and values. When storing a password you must specify unique pairs
> of attributes names and values, and when looking up a password you provide the
> same attribute name and value pairs.

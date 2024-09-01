---
title: Redmined CLI に自身を最新に更新する機能を追加した
---

以前作った [Redmined CLI](2024-04-07-docked-cli-for-redmine-development-environment.md) を日々使い倒している。また、それに合わせて少しづつ改善を進めてきた。

Changelog を書いたり、特段のリリースプロセスを行っているわけではないので、ここで主な変更点をまとめておく。

## Redmined の CLI と Docker image を更新する機能を追加

```
$ redmined -u
```

上記のコマンドを実行すると、次の処理が行われ、手元の Redmined がその時点の最新の状態に更新される。

* `redmined` 自身を https://github.com/hidakatsuya/redmined/blob/main/redmined に差し替える
* ローカルにある Redmined の Docker image を全て `docker pull` して最新に更新する

## 設定ファイル .redmined.json の中身を出力する

```
$ redmined -c
{
  "default": {
    "name": "redmica",
    "port": "3001"
  },
  "ruby3.2": {
    "name": "redmica-ruby3.2",
    "ruby": "3.2"
  }
}
```

`cat .redmined.json` と同じ。

## `docker run` するときの platform、TTYの無効化、

### REDMINED_PLATFORM

```
$ export REDMINED_PLATFORM=linux/amd64
$ redmined bin/rails server
```

設定ファイルでも次のように設定できる。

/path/to/redmine/.redmined.json
```json
{
  "default": {
    "name": "redmine",
    "platform": "linux/amd64"
  }
}
```

### REDMINED_NO_TTY

デフォルトでは `redmined` に渡されたコマンドは `docker run -it` で実行される。
tty の無い環境で `redmined` を実行したい場合は、次のように tty を無効にできる。

```
$ export REDMINED_NO_TTY=1
$ redmined bundle install
```

または、`-T` オプションでも同様のことが実現できる。
```
$ redmined -T bundle install
```

## コマンドヘルプを改善

コマンドヘルプに使えるオプションとコマンドのサンプルを追記した。

```
$ redmined
Usage: redmined [options] [command]

Command:
  Commands to run in the container

Options:
  -n NAME  Specify the configuration name to load from the configuration file
  -T       Run the commands in non-TTY mode
  -c       Print the contents of the configuration file
  -u       Update the redmined script itself and the redmined images to the latest version

Examples:
  Run commands in the container.
  $ redmined ruby -v
  ruby 3.3.4 (2024-07-09 revision be1089c8ec) [x86_64-linux]

  $ redmined bundle install
  Fetching gem metadata from https://rubygems.org/...
  ...

  Enter the container through the bash shell.
  $ redmined bash
  developer@7cc80c9cd1f7:/redmine$

  Launch the Redmine server.
  $ redmined bin/rails server
  => Booting Puma
  => Rails 7.2.1 application starting in development
  => Run `bin/rails server --help` for more startup options
  Puma starting in single mode...
  * Puma version: 6.4.2 (ruby 3.3.4-p94) ("The Eagle of Durango")
  *  Min threads: 0
  *  Max threads: 5
  *  Environment: development
  *          PID: 1
  * Listening on http://0.0.0.0:3000
  Use Ctrl-C to stop

  Run the comands on the "any-config" configuration in the .redmined.json.
  $ redmined -n any-config bundle show

  Run the commands in non-TTY mode.
  $ redmined -T bin/rails test

  Print the contents of the .redmined.json.
  $ redmined -c
  {
    "default": {
      "name": "redmica",
      "port": "3001"
    },
    "ruby3.2": {
      "name": "redmica-ruby3.2",
      "ruby": "3.2"
    }
  }

  Update the redmined script itself and the redmined images to the latest version.
  $ redmined -u
  Updating redmined...
  Installed redmined to /home/hidakatsuya/.local/bin

  Updating redmined images...
  ghcr.io/hidakatsuya/redmined:latest
  ghcr.io/hidakatsuya/redmined:ruby3.3
```

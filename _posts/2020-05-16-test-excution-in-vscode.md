---
title: VSCode の Tasks でテストランナーなどの実行環境を整えた
---

VSCode の標準機能の [Tasks](https://code.visualstudio.com/docs/editor/tasks) を使って、テストランナーなどの実行環境を整えた。

実際の `tasks.json` はこんな感じだ。なお、これは Rails アプリを Docker 上で開発している場合のもの。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "rspec: run active example",
      "type": "shell",
      "command": "docker-compose exec app bin/rspec ${relativeFile}:${lineNumber}",
      "group": "test",
      "presentation": {
        "showReuseMessage": false,
        "focus": true
      },
      "problemMatcher": []
    },
    {
      "label": "rubocop: inspect active file",
      "type": "shell",
      "command": "docker-compose exec app bin/rubocop -D ${relativeFile}",
      "presentation": {
        "showReuseMessage": false,
        "focus": true
      },
      "problemMatcher": []
    }
  ]
}
```

1つ目は、アクティブな spec ファイルのパスとカーソル行数を指定して app コンテナ上で rspec を実行するタスク。2つ目は、アクティブなファイルに対して app コンテナ上で rubocop を実行するタスク。

もし Docker でないなら、例えば1つ目のタスクは、

```json
"command": "bin/rspec ${relativeFile}:${lineNumber}"
```

シンプルにこれだけで良い。

`${relativeFile}` は [変数 (Variable substitution)](https://code.visualstudio.com/docs/editor/tasks#_variable-substitution) というもので、アクティブなファイルのパスなど、いろいろな値を動的に埋め込むためのもの。これを使うと大抵のことはできる。

定義したタスクを実行する場合は、コマンドパレットで `Tasks: Run Task` を選択後、定義したタスクを選択するだけ。実行内容と結果はターミナルに表示される。

手順が面倒？それなら、[タスクごとにショートカットキーを割り当てる](https://code.visualstudio.com/docs/editor/tasks#_binding-keyboard-shortcuts-to-tasks) こともできる。

ちなみに、2つ目の rubocop タスクはほとんど使わない。rubocop を実行する場合は、

```json
{
  "label": "rubocop: inspect changed files to master",
  "type": "shell",
  "command": "git diff --name-only origin/master | xargs docker-compose exec -T app bin/rubocop -aD",
}
```

大抵はこのタスクを使っている。これは、master との差分ファイルに対して rubocop を実行するタスクだ。便利。

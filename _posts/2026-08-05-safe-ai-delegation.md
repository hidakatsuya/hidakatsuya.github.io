---
title: AIに安全に作業を任せる
---

AI（Codex）を使って開発するとき、とにかく承認の対応がめんどくさい。かといって、AI になんでも任せることはしたくない。

その解決のためにしたことをまとめる。

## 基本的な考え方

セキュリティを前提にしつつ、日常的な開発・調査・検証で発生する確認の手間を減らし、極力AIが自律的に作業できるようにする。

- 原則、作業ディレクトリへの書き込みのみ許可し、ネットワークアクセスは拒否する
- 作業ディレクトリ以外への書き込みや、ネットワークアクセスが伴うコマンドのうち「安全だと分かっているもの」だけを明示的に許可する
- AWS CLI や CDK デプロイなど影響が大きいコマンドは、明示的に拒否する
- それ以外のコマンドや sandbox 外の操作は Auto-review に任せる

後から気づいたが、この内容は OpenAI が5月に出した [Running Codex safely at OpenAI](https://openai.com/index/running-codex-safely/) と整合している。必読。

## どう変わったか

一度指示すればタスクが完了するまで承認を求められることは全くなくなった。

もちろんそれは、指示の仕方や、AIが自律的に作業できる環境の提供といった別観点の整備も含めての結果ではある。そのあたりは OpenAI の [Best practices | ChatGPT Learn](https://learn.chatgpt.com/guides/best-practices) も参考になる。

### Auto review とは

Auto-review は、sandbox 外の操作を別のAIエージェントが審査し、安全性を保ちながら人間の承認待ちを減らす仕組み。

[Auto-review | ChatGPT Learn](https://learn.chatgpt.com/docs/sandboxing/auto-review)

実際のチャットではこんな感じになる。

[![](https://github.com/user-attachments/assets/edb646ce-46bd-4c77-94c4-933242d3e263)](https://github.com/user-attachments/assets/edb646ce-46bd-4c77-94c4-933242d3e263)

- メインエージェントが `docker compose exec` を実行しようとする
- `approvals_reviewer (auto_review)` が検証して承認する
- メインエージェントは実行する

なお、ドキュメントにもある通り100%安全ではないので、後述するようなルールをしっかり設定した上で使うもの。OpenAIでも当然実際に使われている。以下、OpenAI のブログより。

- ユーザーの意図、実行環境、セキュリティポリシー、操作の影響を考慮して承認・拒否する
- データ流出、秘密情報の公開、削除、セキュリティ設定の弱体化、信頼できないコードの実行などを防ぐことを重視する
- OpenAI内部では、sandbox 外の操作の約99%を承認し、人間による承認待ちを手動承認モードの約200分の1に削減した
- 頻繁な承認による作業中断と、Full Access や過度に広い許可ルールを使うリスクの間を埋める仕組みとして位置付けられている

## 詳細

実際の設定を示す。

### sandbox・承認・認証・ネットワーク

```toml
# ~/.codex/config.toml

sandbox_mode = "workspace-write"
approvals_reviewer = "auto_review"
cli_auth_credentials_store = "keyring"
mcp_oauth_credentials_store = "keyring"

[sandbox_workspace_write]
network_access = false
```

- 作業領域内への書き込みのみ許可
- sandbox 内からの任意のネットワーク通信はすべて禁止
- sandbox 外の操作は Auto-review の対象
- CLI・MCPの認証情報は keyring に保存（デフォルトだと平文で保存される）

### sandbox外で自動許可するコマンド

```python
# ~/.codex/rules/policy.rules

# git
prefix_rule(pattern = ["git", "fetch"], decision = "allow")
prefix_rule(pattern = ["git", "switch", "-c"], decision = "allow")
...

# gh
prefix_rule(pattern = ["gh", "pr", ["list", "view", "diff", "checks", "status"]], decision = "allow")
prefix_rule(pattern = ["gh", "issue", ["list", "view", "status"]], decision = "allow")
...

# docker
prefix_rule(pattern = ["docker", ["version", "info", "ps", "images", "inspect", "logs"]], decision = "allow")
prefix_rule(pattern = ["docker", ["build", "cp"]], decision = "allow")
...

prefix_rule(pattern = ["redmined", "-T", ["bin/rails", "bundle", "rg", "grep", "ls"]], decision = "allow")
prefix_rule(pattern = ["bundle", "install"], decision = "allow")
...

# 禁止
prefix_rule(pattern = ["cdk", "deploy"], decision = "forbidden")
prefix_rule(pattern = ["npm", "run", "cdk", "--", "deploy"], decision = "forbidden")
prefix_rule(pattern = ["aws"], decision = "forbidden")
```

- 開発でほぼ必ず使うコマンドのうち、**参照系のコマンド**を常に許可する
- 書き込み系のコマンドは Auto review を通す
- 絶対に実行させないコマンド（`cdk deploy` や `aws` など）を禁止する

#### ルールの管理

ルールのファイル構成は次のようにしている。

```shell
~/.codex/rules
├── default.rules
└── policy.rules
```

- **policy.rules**：
すべてのルールを定義する場所。といっても38行程度しかない
- **default.rules**：
基本的に空で、作業中に「常に許可」すると追加される。定期的にこの内容を整理して、必要なら policy.rules に移動する。ただ、承認をする機会がないのであまり活用できていない

### MCPの利用制限

#### 参照系は許可

```toml
# ~/.codex/config.toml

[mcp_servers.aws-documentation-mcp-server.tools.read_documentation]
approval_mode = "approve"
...
```

#### ブラウザへのアクセスは Playwright に限定し、localhost と安全な操作のみ許可

```toml
# ~/.codex/config.toml

[plugins."browser@openai-bundled"]
enabled = false
```

```toml
# ~/.codex/config.toml

[mcp_servers.playwright]
args = [
  "@playwright/mcp@latest",
  "--headless",
  "--allowed-origins",
  "http://localhost:*;http://127.0.0.1:*;http://[::1]:*"
]

[mcp_servers.playwright.tools.browser_click]
approval_mode = "approve"

[mcp_servers.playwright.tools.browser_navigate]
approval_mode = "approve"
...
```

- ChatGPT（Codex, Work）にバンドルされている Browser プラグインを無効化
  - 内部的には Playwright が動作するので十分使えるが、localhost の制御がしづらいため Playwright を採用。こちらは無効にしている
- Playwright は localhost へのアクセスのみ許可する
- 動作検証などで必須の操作のみ明示的に許可し、それ以外は Auto review に任せる

#### GitHub MCP は参照のみで利用

```toml
# ~/.codex/config.toml

[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_TOKEN_MCP_PUBLIC"

[mcp_servers.github.http_headers]
X-MCP-Readonly = "true"
```

- GitHub MCP はコード参照で依然として便利なので、参照のみ許可している
- トークンを読み取り専用にしつつ、念のため `X-MCP-Readonly` も設定している
- GitHub リポジトリの書き込みは `gh` のみで行う

## Codex の設定・ルールの管理も Codex が行う

専用プロジェクトを作って、Codex 運用の基本方針を定義し、その方針に基づいてAIが設定と検証を行う。設定は dotfiles としてバージョン管理し、Codex の設定のみAIが参照・書き込みできるようにしている。

[![](https://github.com/user-attachments/assets/126e8d29-5e34-444e-9597-323a95e1d57a)](https://github.com/user-attachments/assets/126e8d29-5e34-444e-9597-323a95e1d57a)

まず、`mise dotfiles` を使って Codex も含めて dotfiles 管理している。構成はこう。

```shell
❯ tree ~/.dotfiles/.codex
.codex
├── agents
│   ├── executor.toml
│   └── scout.toml
├── AGENTS.md
├── config.toml
├── rules
│   ├── default.rules
│   └── policy.rules
└── skills
    └── orchestrator
        ├── agents
        │   └── openai.yaml
        └── SKILL.md
```

そして、設定を編集するための Codex プロジェクトを作成し、このディレクトリを書き込み許可対象に追加。

```shell
~/dev/projects/codex-config
├── .codex
│   └── config.toml
├── AGENTS.md
├── docs
│   └── policy.md
└── README.md
```

docs/policy.md に方針を明記し、それを AGENTS.md で読ませる。

```markdown
# Codex Harness

## 目的

- セキュリティを前提に、Codex の開発・調査・検証における確認の摩擦を減らし、効率と品質を高める。

## 方針

- 通常の作業ディレクトリへの書き込みを許可し、それ以外の変更・削除は確認を求める。
- 任意の外部ネットワーク通信は許可せず、用途と影響範囲が既知のコマンドだけを許可する。
- 自動許可するコマンドは、開発で繰り返し使い、影響範囲を理解できるものだけに絞る。

## 実現する設定

- `~/.codex/config.toml`: sandbox とネットワークの設定。
- `~/.codex/rules/policy.rules`: sandbox 外で実行可能なコマンドを定義する。
- `~/.codex/rules/default.rules`: 追加許可されたルールを記録し、その内容を基に `policy.rules` を改善する。
- `~/.codex/AGENTS.md`: rules で定義できない作業上のルールを定義する。

## 実装・設計時のルール

- 実際の設定は `config.toml` と `rules/*.rules` を正本とし、この文書へ設定値やコマンド一覧を重複して書かない。
- 権限、ネットワーク、または許可コマンドを広げる前に、より狭い代替案を検討する。
- ルール変更後は `codex execpolicy check` と Codex Desktop の実動で検証する。
```

`.codex/config.toml` で書き込み場所に dotfiles のディレクトリを追加定義。

```toml
[sandbox_workspace_write]
writable_roots = ["/Users/hidakatsuya/.dotfiles/.codex"]
```

## 最後に

とにかく難しいことはしたくないし、設定もたくさん書きたくなかったので、Codex が提供する設定と機能だけを使い、ベストプラクティスに限りなく従う形で対応した。結果的に、最小限かつ十分実用的で長く使えそうな環境ができたと思う。Auto-review が便利すぎる。

それと、現在の開発では、gpt-5.6 Sol を orchestrator として、Terra の executor サブエージェントと、同じく Terra の scout サブエージェントで開発を行う形をとっている。orchestrator は、要件の定義とタスクの分解および指示、成果物のレビューとセキュリティレビューを行う。設定・構成自体は非常にシンプルだが、悪くないような気はしている。その辺もどこかでまとめようと思う。


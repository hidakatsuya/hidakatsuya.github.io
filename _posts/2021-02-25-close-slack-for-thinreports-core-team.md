---
layout: post
title: コアチームの Slack を閉じてコミュニケーションポリシーを見直した
---

2020年11月、[Thinreports](https://github.com/thinreports) コアチームの Slack を閉じた。

## Slack for Thinreports

元々、Thinreports の Slack は、コミュニティの議論の場とするつもりで開設した。しかし、そうなる(する)こともなく、いつの間にか @maeda-m と二人だけのコアメンバー専用として、そのまま運用されてきてしまった。

今考えると、これは本当に良くなかった。

仕様決定のプロセスや議論が見えないとか、ロードマップが見えないなど、コミュニティにとっていくつもの悪影響が考えられるが、何より、コミュニティが議論や開発に参加する機会を奪ってしまっていたことが本当に良くない。しかも数年も。

その状況を改善すべく、昨年11月に Slack を閉じ、今後のコミュニケーションポリシーを策定することにした。

## Communication Policy

![](/images/2021-02-25-communication-policy.png)

策定したポリシーは次のようなもの。

1. 原則、全ての議論、全ての作業は Issue、Pull request、[Discussions](https://github.com/thinreports/thinreports/discussions) のいずれかで行う
2. センシティブな情報を伴う議論のみ [Team Discussions](https://docs.github.com/ja/github/building-a-strong-community/about-team-discussions) で行う
3. 日本語OK

### Issue, Pullrequest, Discussions

ポリシー策定後は、[GitHub Discussions #13](https://github.com/thinreports/thinreports/issues/13) や [Renewal rails example #17](https://github.com/thinreports/thinreports/issues/17) のように、Issue 上で議論するようにしている。

実は、以前下記の記事にまとめたリリースプロセスの整備も、このコミュニケーション改善の一貫だった。

[一年半ぶりに Thinreports v0.11.0 をリリースして、リリース手順などの運用周りを見直した](2020-06-20-thinreports-v0.11.0-released.md)

### Team Discussions

例えば Organization の権限や設定、運用についての議論や、RubyGems.org などのアカウント周りの話題など、どうしても public な場で話すことができない議論もある。そういったセンシティブな情報を扱う議論は、 `thinreports/core` チームの Team Discussions で行う。

![](/images/2021-02-25-sensitive-discussion.png)

### 日本語OK

英語が議論の妨げになるぐらいなら、日本語でガンガン議論する方が良いよね、というもの。

Thinreports は少なからず海外のユーザもいるので、英語で議論することが望ましい。しかし、より良い議論を行い、良いプロダクトを作ることの方が今の Thinreports Organization には必要だと考えた。

![](/images/2021-02-25-japanese-ok.png)

## 最後に

現在、v1.0 に向けたロードマップとタスクを GitHub Projects を使って整理しようとしている。もちろん、その Project は public だ。また、タスクの中には、比較的簡単なものもある。そういったものは [good first issue](https://github.blog/2020-01-22-browse-good-first-issues-to-start-contributing-to-open-source/) ラベルを付けて、より開発に参加しやすいようにしていきたい。

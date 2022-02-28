---
title: 120個程の Vue2 コンポーネントを Composition API に一括移行した
---

[Thinreports Section Editor](https://github.com/thinreports/thinreports-section-editor) の Vue3 移行を進めている。
その最初のステップとして、Vue2 (Options API) コンポーネントを [Composition API](https://github.com/vuejs/composition-api) へと移行した。

コンポーネントは全部で 120 個程あるため、移行スクリプトで一括移行した。

- [実際の Pullrequest](https://github.com/thinreports/thinreports-section-editor/pull/22)
- [移行スクリプト (Gist)](https://gist.github.com/hidakatsuya/f1faf5753b6eddf9cf4bc4dae276f181)

最初は、移行スクリプトを書くのが面倒で、定型の構文変換だけをスクリプトで行い、大部分は手動で修正するつもりだった。のだが、3ファイル程修正したところで早々に心が折れてしまった。
結果的には、だいぶ雑ではあるが、スクリプトを実装する時間も含めて、2日程度で移行することができた。

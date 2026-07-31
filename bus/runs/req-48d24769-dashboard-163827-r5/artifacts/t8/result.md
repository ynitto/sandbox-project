# t8 成果報告

## 成果／サマリー

- `tools/agent-dashboard/src/features/agent-project/main/project.js` は変更不要と確認した。
- uxreview の2指摘は、いずれも同ファイルが利用する共有設定リーダー `main/toolconfig.js` の解析・探索処理が根本原因である。
  - malformed local JSON を無視して global 設定へ fallback する問題
  - YAML folded scalar の空行段落を `body.join(' ')` で潰す問題
- `project.js` の `consistencyGateStatus()` は `readToolConfig()` の返値を読み取り専用の表示状態へ変換する責務であり、ここで再解析や探索補正を追加すると根本原因を残した重複実装になるため変更していない。
- 作業ツリー内のファイルは書き換えていない。

## 検証内容と結果

- `git diff --check`: 成功。
- `node tools/agent-dashboard/test/consistency-gate.test.js`: 10件成功。
- `tools/agent-dashboard` で `npm test`: 成功（終了コード0）。
- `git status --short`: 出力なし。作業ツリーは clean。

## 前提・未解決事項・範囲外

- 前提: `[scope]` は指定された `project.js` のみを変更可、`[out_of_scope]` と「変更してよいのは tools/agent-dashboard 配下」のうち、より狭い制約を優先した。
- 前提: 「根本原因で修正」は、共有リーダーの欠陥を `project.js` 側で症状回避しないことを含むと解釈した。
- 未解決: uxreview の2件は残っており、`tools/agent-dashboard/src/features/agent-project/main/toolconfig.js` と対応する回帰テストを担当する別タスクで修正が必要。
- 範囲外で見つけた問題: 上記2件のみ。renderer、agent-project 本体、状態書換経路には触れていない。

@followup `toolconfig.js` で最初に見つかった malformed JSON を解析失敗として返し、global fallback を止める回帰修正を行う。
@followup `parseFlatYaml()` の folded scalar で空行段落を保持し、PyYAML と一致する回帰テストを追加する。

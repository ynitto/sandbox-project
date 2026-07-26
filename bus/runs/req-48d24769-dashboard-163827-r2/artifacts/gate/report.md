# agent-dashboard gate verification

## 判定

`verify=pass`

## 独立検算

- コンフリクト: `git ls-files -u` と conflict marker 検索の双方で残存なし。
- スコープ: 差分は `tools/agent-dashboard/` 配下の7ファイルのみ。
- regression/intake: 設定有無・コマンド値・codd-gate 結線を別々に表示し、公式
  `codd_gate_wiring.py` と同形の正規表現で判定。
- 未結線時: README と同形の2設定行、設定ファイルを開く導線、regression 用 sibling CLI、
  intake は直接編集である旨を表示。UI 自身は設定を書き換えない。
- needs/inbox/commands: needs は構造化 frontmatter を読み、既存の commands 投函・inbox 取込契約を
  変更していない。
- done 不変条件: ゲート UI の唯一の操作は `api.openPath`。done や agent-project 状態への直接書込なし。
- 可読性: 検証失敗の要約・context・判断材料の折り畳みを維持し、intake は「正常実行時」にだけ
  起票されると明示。

## 検出・修正した不適合

- `src/renderer/sections/needs.js` の旧記録フォールバックが現在の
  `regressionWired` に依存し、失敗後に未結線化すると、記録中に `回帰検知` と
  `codd-gate` 実コマンドが残っていても回帰経路・対処導線を消していた。
- 実行時記録の `回帰検知` + `codd-gate` を根拠に分類する最小修正へ変更し、
  現在未結線の回帰失敗を固定する回帰テストを追加。

## agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---:|---:|---:|---:|
| functional（修正後再レビュー） | LGTM | 0 | 0 | 0 |
| ai-antipattern | LGTM | 0 | 0 | 0 |
| architecture | LGTM | 0 | 0 | 1 |
| test | LGTM | 0 | 0 | 0 |

総合判定: LGTM

## 検証結果

- `npm test`: exit 0（全件成功）
- 対象5テスト: exit 0
  - consistency-gate: 7/7
  - consistency-gate-ui: 成功
  - needs-gate-integration: 8/8
  - needs-diagnosis: 13/13
  - overview-ui: 成功
- `git diff --check`: exit 0

## 非 blocking follow-up

`@followup tools/agent-project/agent_project/mr.py の regression 失敗 _block 呼出しで
failure-phase: regression と failure-command を構造化出力し、renderer の散文フォールバック依存を解消する。`


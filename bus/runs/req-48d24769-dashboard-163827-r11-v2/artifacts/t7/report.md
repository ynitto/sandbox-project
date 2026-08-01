# 未設定コマンドの有効化導線

## 成果／サマリー

`tools/agent-dashboard/src/renderer/renderer.js` の一貫性ゲート案内を、`regression_cmd` または `intake_cmd` が未設定の場合だけ表示するよう修正した。

- 設定例には未設定のキーだけを含める。
- 両キーが設定済みなら、codd-gate に未結線でも設定編集ボタンと sibling CLI を表示しない。
- 未設定の `regression_cmd` には README と同じ設定行と `tools/agent-project/` の `codd_gate_regression.py` を案内する。
- 未設定の `intake_cmd` には README と同じ設定行と直接編集を案内する。
- 結線済み／未結線、設定あり／なし、既存コマンドの表示は維持し、UI から設定や状態を書き換える処理は追加していない。

`test/consistency-gate-ui.test.js` に、設定済みキーの置換例を出さないことと、両キー設定済みなら未結線でも有効化導線を出さないことの回帰確認を追加した。

## 検証内容と結果

- `npm test`: PASS（agent-dashboard 全テスト）。
- `npx eslint src/renderer/renderer.js test/consistency-gate-ui.test.js`: PASS。
- `node --check src/renderer/renderer.js`: PASS。
- `git diff --check -- tools/agent-dashboard`: PASS。
- 変更は `tools/agent-dashboard/src/renderer/renderer.js` と `tools/agent-dashboard/test/consistency-gate-ui.test.js` のみ。commit / push / branch 操作は未実施。

## 採用した前提・未解決事項・範囲外

- 完了条件を「codd-gate への結線有無ではなく、`regression_cmd` / `intake_cmd` の設定有無を有効化導線の表示条件とし、未設定キーだけを README 準拠で案内すること」と解釈した。既存の別用途コマンドを置換へ誘導しないためである。
- `regressionConfigured` / `intakeConfigured` は main 側が公式設定キーから生成する既存表示モデルを正として利用した。新しい設定探索や推測は追加していない。
- agent-project 本体のフック、自動設定、done／タスク状態の書換え、公式 needs/inbox/commands 以外への書込みは範囲外として実施していない。
- 未解決事項および範囲外で新たに見つけた問題はない。

```json
{"ok": true, "issues": []}
```

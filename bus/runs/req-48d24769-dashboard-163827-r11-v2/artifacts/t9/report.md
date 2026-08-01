# overview UI 最小回帰テスト

## 成果／サマリー

`tools/agent-dashboard/test/overview-ui.test.js` の既存一貫性ゲートケースを最小限補強した。

- `regression_cmd` / `intake_cmd` の設定あり・なし表示を既存ケースで検証する。
- 全結線・一部結線・未結線の意味と、設定済みの別コマンドが codd-gate 結線ではないことを検証する。
- 未設定キーだけに README と同じ YAML 設定例を出し、設定済みキーを置換へ誘導しないことを検証する。
- regression 未設定時は設定編集と `tools/agent-project/` の sibling CLI を案内し、両キー設定済みなら未結線でも有効化導線を出さないことを検証する。

変更は指定された `tools/agent-dashboard/test/overview-ui.test.js` のみ。production code、needs-diagnosis テスト、外部 CLI の実行には触れていない。

## 検証内容と結果

- `node --check test/overview-ui.test.js`: PASS。
- `npx eslint test/overview-ui.test.js`: PASS。
- `git diff --check -- tools/agent-dashboard/test/overview-ui.test.js`: PASS。
- 依存 t7（commit `2a0b2eb0589fbe0638758d1db556bb453c911d20`）の `consistencyGateHtml()` を読み取り専用で差し込んだ `overview-ui.test.js`: PASS。
- 現在の worktree HEAD (`59ccf49e71d41fe35533ae718f010649863bd6a3`) 単体の `node test/overview-ui.test.js`: 想定どおり FAIL。t7 がまだ HEAD に統合されておらず、両キー設定済みでも旧実装が有効化導線を表示するため。依存統合後の上記検証では PASS している。

## 採用した前提・未解決事項・範囲外

- 完了条件を、概要 UI の表示契約について「設定状態」「ゲート状態と失敗の意味」「README 準拠の設定編集／sibling CLI 導線」を少数ケースで固定することと解釈した。
- README 準拠は、README の正典である 2 つの YAML コマンド文字列と `<root>/repos.json` プレースホルダが画面に一致すること、とした。
- t7 は依存成果として後続で統合される前提。現 worktree への cherry-pick／checkout は git 利用規約により行っていない。
- 未解決事項および範囲外で新たに見つけた問題はない。

```json
{"ok": true, "issues": []}
```

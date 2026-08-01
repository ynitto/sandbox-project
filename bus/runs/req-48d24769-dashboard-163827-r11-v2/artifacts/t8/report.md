# needs-diagnosis 回帰テスト

## 成果／サマリー

`tools/agent-dashboard/test/needs-diagnosis.test.js` の一貫性ゲート回帰失敗ケースを強化した。

- 追加ゲート表示の由来判定に必要な `failureContext.command` に `codd-gate verify` が残ることを検証。
- 従来の読める回帰失敗要約 `failureSummary` がコマンドと先行工程の成功を保つことを継続検証。
- 要約で隠されてはいけない生の `why` / `detail` も同時に保持されることを検証。
- `blocked` / 未決着の状態保持は既存アサーションで継続検証。

production code、overview UI テスト、状態変更機能は変更していない。

## 検証内容と結果

- `node test/needs-diagnosis.test.js`: PASS（12 tests）。
- `npx eslint test/needs-diagnosis.test.js`: PASS。
- `npm test`: PASS（agent-dashboard 全テスト）。
- `git diff --check -- tools/agent-dashboard`: PASS。
- diff 確認: リポジトリ内の変更は `tools/agent-dashboard/test/needs-diagnosis.test.js` のみ（6 additions, 2 deletions）。commit / push / checkout は未実施。

## 採用した前提・未解決事項・範囲外

- 完了条件を「データ層でゲート由来を判定できる `failureContext.command` と、従来の診断要約・生の判断材料が共存し、回帰失敗が done 扱いにならないこと」と解釈した。表示 DOM 自体は既存の `needs-gate-integration.test.js` が担うため、指定スコープの parseNeeds 回帰検証に限定した。
- 依存成果 t7 の有効化導線変更は overview 側であり、本タスクでは変更もテスト追記もしていない。ただし全テストで共存を確認した。
- 未解決事項および範囲外で新たに見つけた問題はない。

```json
{"ok": true, "issues": []}
```

# t4 調査報告

## (a) 成果／サマリー

完了条件を「概要またはプロジェクト情報、needs-diagnosis、codd-gate、回帰失敗要約の現行経路を追跡し、`regression_cmd` / `intake_cmd` の設定有無を、agent-project 本体や UI 書換契約を増やさず dashboard へ渡す最小結合点を確定する」と解釈した。

結論: 最小結合点は既存の `readProject()` スナップショットであり、現行ブランチにはすでに実装済みのため追加変更は不要。

- 設定: `.agents/agent-project.{yaml,yml,json}` 等の公式トップレベルキー `regression_cmd` / `intake_cmd` → `readToolConfig()` → `consistencyGateStatus()` → `readProject().consistencyGate`。
- 配送: `dashboard:project` IPC → preload の既存 `readProject()` → renderer の `state.project`。新規 IPC、needs/inbox/commands 用の別契約は不要。
- 概要: `consistencyGateHtml(state.project)` が `regressionConfigured` / `intakeConfigured` と、codd-gate への `regressionWired` / `intakeWired` を分けて表示する。未結線時は設定編集と既存 sibling CLI の案内だけを出し、書き換えない。
- needs / 回帰失敗: 公式 `needs/<id>.md` の `failure-*` frontmatter → `parseNeeds()` → `readProject().needs` → `needFailureViewModel()` / `renderNeedFacts()`。記録済み phase・要約・実コマンドから codd-gate 由来と確認できる場合だけ、同じ `state.project.consistencyGate` を参照する。
- inbox: `readProject().inboxFiles` は取り込み待ちファイル一覧として同じスナップショットに載るが、ゲート設定の運搬には使わない。
- commands: `commands/*.err` は `listCommandFailures()` で task id ごとに needs へ付与される。操作要求の drop/失敗通知という既存責務を維持し、ゲート設定や done 状態を載せない。

したがって変更点を表すなら、既存 `readProject()` の返却値へ読み取り専用の `consistencyGate` を 1 フィールド載せる一点だけで足りる。現行ブランチはその形になっている。リポジトリ内のファイルは変更していない。

## (b) 検証内容と結果

- `node test/consistency-gate.test.js`: 15 passed。
- `node test/consistency-gate-ui.test.js`: success。
- `node test/needs-gate-integration.test.js`: 8 passed。
- `node test/needs-diagnosis.test.js`: 12 passed。
- `node test/needs-command-failure.test.js`: 4 passed。
- `node test/overview-ui.test.js`: success。
- `npm test`: 成功（exit 0）。
- `git diff --check`: 成功。
- 作業ツリー: ソース変更なし。

## (c) 前提・未解決事項・範囲外

- 前提: `dashboard:project` の既存スナップショット返却を公式 dashboard 契約、`regression_cmd` / `intake_cmd` と needs の `failure-*` を公式 agent-project 契約として扱った。
- 前提: 「設定有無」は空でない実効値の有無、「結線有無」は codd-gate の正規コマンド形への一致であり、両者を混同しない。
- 未解決: agent-project が `--config` で明示した実効設定パスは instance/status 契約に無いため、dashboard の自動探索結果との一致は断定できない。画面もこの制約を明示している。
- 範囲外: agent-project 本体フック、UI からの設定・task/done 書換、codd-gate の実在・互換性・実行成功確認には手を加えていない。
- @followup: 実効 `--config` との一致を将来保証するなら、dashboard 独自推測ではなく agent-project の公式 instance/status 契約へ解決済み設定パスを追加する別タスクが必要。

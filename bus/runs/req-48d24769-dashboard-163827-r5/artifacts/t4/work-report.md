# t4 agent-dashboard main データモデル

## 成果／サマリー

完了条件を「`tools/agent-dashboard/src/features/agent-project/main/project.js` の既存 `readProject()` ペイロードへ、実効 `agent-project` 設定にある `regression_cmd` / `intake_cmd` の設定有無と codd-gate 結線状態を読み取り専用で載せること」とした。

対象ブランチの対象ファイルには既に次の最小実装が存在し、依存成果 t3 および公式 `tools/agent-project/codd_gate_wiring.py` の契約と一致していたため、追加編集は行っていない。

- `consistencyGateStatus(projectCfg)` が `configFile`、`regressionConfigured`、`intakeConfigured`、`regressionWired`、`intakeWired`、`wired`、表示用の両コマンド値を生成する。
- 結線判定は公式実装と同じ語順条件（regression: `codd-gate ... verify ... --base`、intake: `codd-gate ... tasks ... --debt`）を使う。汎用フックに別コマンドが設定された場合は「設定あり・未結線」と区別する。
- `readProject()` は既存の実効設定 `projectCfg` を再利用し、返却スナップショットの `consistencyGate` に状態を載せる。設定・状態への書き込み経路は追加しない。

## 検証内容と結果

- `node test/consistency-gate.test.js`: 10 passed。
- `npm test`: exit 0、全テスト成功（`consistency-gate-ui.test.js`、`needs-gate-integration.test.js` を含む）。
- `git diff --check`: 成功。
- `git status --short` / `git diff --name-only`: 出力なし。作業ツリーは clean で、本タスクによるリポジトリ変更なし。

## 前提・未解決事項・範囲外

- 前提: 現ブランチ HEAD に既にある対象実装を本タスクの成果として検証し、同じ実装を重ねて変更しないのが最小差分である。
- 前提: 「設定有無」は空文字・空白・YAML null を未設定とし、文字列値があれば codd-gate 以外でも設定済みとする。
- 未解決事項なし。
- agent-project 本体フック、done 状態、renderer UI、設定保存 API は変更していない。


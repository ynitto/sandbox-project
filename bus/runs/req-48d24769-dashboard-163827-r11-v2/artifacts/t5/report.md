# overview/project-info 一貫性ゲート応答

## 成果／サマリー

コード変更は不要と判断した。HEAD `59ccf49e7` の `tools/agent-dashboard/src/features/agent-project/main/project.js` は、既存の `readToolConfig('agent-project', ...)` 経路を一度だけ使い、`readProject()` 応答の `consistencyGate` に次を既に返している。

- `regressionConfigured` / `intakeConfigured`: `regression_cmd` / `intake_cmd` の空でない設定有無
- `regressionWired` / `intakeWired`: codd-gate 正規形への個別結線状態
- `wired`: 両方の結線状態を main 側で一度だけ合成した値
- `regressionCmd` / `intakeCmd`: 別コマンド設定済みと未設定を UI が区別するための読み取り値
- `configFile` / `configError` / `configSource` / `explicitConfigUnknown`: 自動探索した設定元と判定限界

この応答は設定・タスク状態・done 状態を書き換えず、UI 側の既存有効化導線（設定編集／sibling CLI）が必要とする情報を提供する。重複する再実装は加えず、作業ツリーの追跡差分は 0 件。

## 検証内容と結果

- `npm test`: PASS（agent-dashboard 全テスト）。
- `node test/consistency-gate.test.js`: PASS（15件）。設定有無、片側設定、非 codd-gate 値、YAML scalar、設定探索、壊れた JSON を確認。
- `node test/consistency-gate-ui.test.js`: PASS。
- `node test/needs-gate-integration.test.js`: PASS（8件）。
- `node --check src/features/agent-project/main/project.js`: PASS。
- `npx eslint src/features/agent-project/main/project.js test/consistency-gate.test.js`: PASS。
- `git diff --check -- tools/agent-dashboard`: PASS。
- `git status --short`: 出力なし。

初回の `npm test` は worktree に `yaml` が未導入で停止した。`package.json` の既定依存を `npm install --ignore-scripts --no-audit --no-fund` で導入後に再実行し、全件成功した。`node_modules/` と `package-lock.json` は既存 `.gitignore` 対象で、追跡差分は生じていない。

## 採用した前提・未解決事項・範囲外

- 完了条件を「一覧 discovery ではなく、概要画面が取得する完全な project-info 応答 `readProject()` に、設定有無と結線判定を読み取り専用で一貫して供給すること」と解釈した。依存成果 t2 の特定経路および現物コード・テストが一致した。
- `regression_cmd` / `intake_cmd` は汎用フックなので、「設定済み」と「codd-gate 結線済み」を別フラグにする現行契約を維持した。
- agent-project 起動時の明示 `--config` は dashboard から判別できない。現行応答は `explicitConfigUnknown: true` として断定を避ける。
- UI 描画、agent-project 本体、done 状態変更コマンドには触れていない。
- @followup 明示 `--config` の実効設定元を正確に表示する必要がある場合は、agent-project の instance/status 読み取り契約へ設定ソースを追加する別タスクとする。

```json
{"ok": true, "issues": []}
```

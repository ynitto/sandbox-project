# configured-but-unwired guidance 修正報告

## 成果

- `contractgate/report.md` と `uxgate/report.md` の同一根因を確認し、`consistencyGateHtml()` の有効化判定を「キー未設定」から `regressionWired` / `intakeWired` の「未結線」へ変更した。
- 別用途の `regression_cmd` / `intake_cmd` が設定済みでも、未結線キーには README 準拠の設定例、設定ファイルを開く導線、既存処理を失う置換警告を表示する。
- sibling CLI は README と同じく未設定の `regression_cmd` にだけ案内する。設定済み値を UI や CLI から自動置換しない。
- `overview-ui.test.js` と直接関連する `consistency-gate-ui.test.js` の逆向き期待値を更新した。production の変更は renderer の読み取り専用表示だけで、needs/inbox/commands や状態書換経路には触れていない。

## 検証

- 指定 grep: PASS（exit 0）
- `node tools/agent-dashboard/test/needs-diagnosis.test.js`: PASS（12 tests、`failureSummary` / `why` / `detail` / `failureContext.command` を保持）
- `node tools/agent-dashboard/test/overview-ui.test.js`: PASS
- `node tools/agent-dashboard/test/consistency-gate-ui.test.js`: PASS
- 対象4ファイル ESLint: PASS
- `git diff --check`: PASS
- `tools/agent-dashboard` の `npm test`: PASS

## 前提・未解決事項・範囲外

- 前提: `wired=false` は、設定値の有無ではなく codd-gate への未結線を示す正準状態として扱った。設定済みキーでは sibling CLI による自動更新を案内せず、手動で置換／併合を判断させるのが UX gate の安全要件と解釈した。
- 未解決事項: なし。
- 範囲外で見つけた問題: なし。`agent-project` 本体、done 不変条件、公式契約外の書込先は変更していない。

```json
{"ok": true, "issues": []}
```

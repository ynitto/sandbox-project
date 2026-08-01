# overview/project-info 一貫性ゲート描画

## 成果／サマリー

コード変更は不要と判断した。HEAD `59ccf49e7` の `tools/agent-dashboard/src/renderer/renderer.js` と `src/renderer/sections/overview.js` には、指定された表示と読み取り専用の有効化導線が既に実装されている。

- `consistencyGateHtml(p)` が `regression_cmd` / `intake_cmd` ごとに「設定: あり／なし」と「結線済み／未結線」を別々に表示する。
- セクション見出しは一貫性ゲート全体を「結線済み／一部結線／未結線」の三状態で表示する。
- 既存 UI の `badge info` / `badge warn`、`need-resolution`、`summary-actions`、`summary-link secondary` を再利用している。
- 未結線時だけ README と同じ設定例を示し、条件を満たす場合は `regression_cmd` 用 sibling CLI、`intake_cmd` の直接編集、自動検出した設定ファイルを開く導線を表示する。
- 設定ファイルを開くボタンは `api.openPath()` のみを呼び、設定値、needs、タスク状態、done 状態を書き換えない。

重複実装を避けるため、`tools/agent-dashboard` 配下を含め作業ツリーは変更していない。

## 検証内容と結果

- `node test/consistency-gate-ui.test.js`: PASS。設定有無、全結線／部分結線／未結線、別コマンド設定、YAML/JSON 導線、XSS エスケープを確認。
- `node test/overview-ui.test.js`: PASS。overview への描画結線を確認。
- `node --check src/renderer/renderer.js`: PASS。
- `node --check src/renderer/sections/overview.js`: PASS。
- `npx eslint src/renderer/renderer.js src/renderer/sections/overview.js test/consistency-gate-ui.test.js`: PASS。
- `npm test`: PASS（agent-dashboard 全テスト）。
- `git diff --check -- tools/agent-dashboard`: PASS。
- `git status --short`: 出力なし。

## 採用した前提・未解決事項・範囲外

- 完了条件を「overview の既存 UI パターンで、両コマンドの設定有無と codd-gate 結線状態を区別し、未結線時に人が有効化できる読み取り専用導線を示すこと」と解釈した。依存成果 t3/t5 と現物コード・テストが一致した。
- `regression_cmd` / `intake_cmd` は汎用フックであるため、「設定済み」と「一貫性ゲートへ結線済み」は別状態として表示する現行仕様を維持した。
- dashboard が自動探索した設定候補と、agent-project が明示 `--config` で利用する設定が一致するかは断定できない。現行 UI はこの制約を明記している。
- 設定変更ボタン、done 状態の書換え、needs-diagnosis 表示、agent-project 本体のフック実装は範囲外として変更していない。
- 範囲外で追加修正が必要な問題は見つからなかった。

```json
{"ok": true, "issues": []}
```

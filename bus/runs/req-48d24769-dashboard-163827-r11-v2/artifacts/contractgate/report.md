# t5〜t9 dashboard 契約検証

verify=fail

## 独立検証結果

- 現 HEAD `98cb4584a` は t7 (`2a0b2eb05`)、t8 (`86699a1e5`)、t9 (`98cb4584a`) をすべて含む。t5 は変更なし。
- t5 開始点 `59ccf49e7` からの dashboard 差分は4ファイル、41 additions / 19 deletions。production 差分は `src/renderer/renderer.js` のみで、残り3ファイルはテスト。`tools/agent-dashboard` 外の差分はない。
- 依存導入後の dashboard 全 `npm test` は PASS。変更4ファイルの ESLint・構文検査、`git diff --check` も PASS。worktree の追跡差分はない。
- 全体 `npm run lint` は今回の差分外に既存4エラーがあり FAIL したが、変更4ファイルだけの ESLint は PASS のため本判定の issue には数えない。

## 公式契約との照合

- ゲート表示は公式設定キー `regression_cmd` / `intake_cmd` を `readToolConfig()` → `consistencyGateStatus()` → `readProject().consistencyGate` で読む。新しい非公式な表示データ源はない。
- needs の `failure-phase` / `failure-summary` / `failure-command` は本体 `agent_project/needs.py` の `_FAILURE_FM_KEYS` と一致し、`parseNeeds()` 後も `why` / `detail`、blocked・未決着状態を保持する。
- 今回の production 差分は renderer の表示条件だけ。needs の Decision Outcome、inbox JSON、commands JSON の既存書込み関数・IPC・preload は無変更で、task/backlog status や `done` の直接変更、新しい書込み経路、外部 CLI 実行は追加されていない。設定ファイルのボタンも既存 `openPath` で開くだけである。
- README の正規設定例と画面のコマンド文字列は一致する。

## issues

1. `tools/agent-dashboard/src/renderer/renderer.js:1232-1252` は有効化導線を `regressionConfigured` / `intakeConfigured` の欠落で判定する。このため両キーに別コマンドが設定済みで `regressionWired=false` / `intakeWired=false` の場合、画面は「未結線」と表示する一方、README 準拠の設定編集・sibling CLI 導線をすべて隠す。元要求と dashboard README の「未結線なら」有効化手順に反する。`regressionWired` / `intakeWired` を基準に未結線キーの設定例と導線を出し、既存値は自動変更せず、置換すると現在の処理が失われる警告を復元すること。`test/consistency-gate-ui.test.js:136-153` と `test/overview-ui.test.js:345-352` の「設定済みなら導線を出さない」期待値も同時に直すこと。

{"ok": false, "issues": ["tools/agent-dashboard/src/renderer/renderer.js:1232-1252 が、設定済みだが codd-gate 未結線のとき README 準拠の有効化導線を隠す。wired フラグ基準で導線を出し、既存コマンド置換の警告を表示し、対応テストを修正すること。"]}

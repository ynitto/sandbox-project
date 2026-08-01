# UX gate verification

verify=fail

## Blocking issue

- `tools/agent-dashboard/src/renderer/renderer.js:1232-1252`: `regression_cmd` と `intake_cmd` の両方に別用途コマンドが設定済みで、codd-gate は両方未結線のとき、`settings` が空になり有効化手順・設定ファイルを開く導線がすべて消える。画面は「未結線」「一貫性ゲートの検査ではありません」と示すだけで対処不能になり、元要求の「未結線時は README と同じ有効化導線」に反する。`needs.js:1488` の「概要タブで有効化」案内先も行き止まりになる。未結線時は導線を表示し、設定済みキーには既存処理を失う可能性と置換／併合判断、README 準拠の例、設定ファイルを開く操作を示すこと。sibling CLI は安全上 `regression_cmd` 未設定時だけでよい。
- `tools/agent-dashboard/test/consistency-gate-ui.test.js:136-153`、`tools/agent-dashboard/test/overview-ui.test.js:345-352`: 上記の誤動作を「有効化導線を出さない」正解として重複固定している。未結線なら有効化・設定ファイルを開く導線・README 準拠例・設定済みキーの置換警告が出る期待へ修正すること。

## Independent checks

- `main...HEAD`: 4 files, 41 insertions, 19 deletions。差分は `tools/agent-dashboard` 配下のみ。
- dashboard 全テスト: PASS（依存 `yaml` が未導入だったため `npm install --no-package-lock --ignore-scripts` 後に再実行）。
- 対象4ファイル ESLint: PASS。
- `git diff --check`: PASS。
- needs diagnosis: `failureContext.command`、`failureSummary`、`why`、`detail`、`blocked`、未決着状態の保持を確認。
- t7〜t9 の3コミットは現ブランチに存在。t6 の個別成果物／コミットは提示範囲になかったため、`main...HEAD` 全差分で取りこぼしを防いだ。
- スコープ外変更なし。レビューのみでソース変更なし。

## agent-reviewer aggregate

| perspective | verdict | Critical | Warning | Suggestion |
|---|---|---:|---:|---:|
| functional | REQUEST_CHANGES | 0 | 1 | 0 |
| ai-antipattern | LGTM | 0 | 0 | 0 |
| architecture | LGTM | 0 | 0 | 0 |
| test | REQUEST_CHANGES | 0 | 1 | 0 |

同一根因を functional と test が検出したため、issues は1件へ集約した。

```json
{"ok": false, "issues": ["tools/agent-dashboard/src/renderer/renderer.js:1232-1252 で、両キーが別用途コマンドとして設定済みかつ codd-gate 未結線の場合に有効化導線が消える。未結線時は設定編集・README準拠例・設定ファイルを開く導線を表示し、test/consistency-gate-ui.test.js:136-153 と test/overview-ui.test.js:345-352 の逆向き期待値も修正すること"]}
```

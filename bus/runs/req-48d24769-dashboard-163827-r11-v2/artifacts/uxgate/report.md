# UX gate 検証結果

verify=fail

## 判定

`regression_cmd` / `intake_cmd` の両方に codd-gate 以外のコマンドが設定済みで、両方とも未結線のケースでは、画面に「未結線」「一貫性ゲートの検査ではありません」と表示される一方、有効化ブロック、README 準拠の設定例、設定ファイルを開く導線、sibling CLI がすべて消える。needs 側は「概要タブの『一貫性ゲート』で有効化してください」と案内するため、利用者は概要へ移動しても対処できず行き止まりになる。

## issues

1. `tools/agent-dashboard/src/renderer/renderer.js:1232-1254`
   - `settings` を `!regressionConfigured` / `!intakeConfigured` で作り、`settings.length === 0` のとき `enable` を非表示にしている。このため「設定あり・未結線」では有効化手順が出ない。
   - 未結線を基準に README の正準値と設定編集導線を表示し、既存の別コマンドを置換すると現在の処理が失われる旨を明記すること。sibling CLI は意図しない上書きを避けるため、現状どおり `regression_cmd` 未設定時だけ提示してよい。
   - `tools/agent-dashboard/test/consistency-gate-ui.test.js:139-155` と `tools/agent-dashboard/test/overview-ui.test.js:345-353` の「両キー設定済みなら導線なし」という期待値を、未結線時には安全な手動編集導線が残る期待値へ修正すること。併せて両コマンド値・`設定: あり` 2件・両行の非 codd-gate 理由を固定すること。

2. `tools/agent-dashboard/test/consistency-gate-ui.test.js:139-155`、`tools/agent-dashboard/test/overview-ui.test.js:345-353`
   - 重点ケースで `intakeCmd` の表示、`設定: あり` が2件あること、非 codd-gate 理由が両行に付くことを検証していない。intake 行だけ消える／未設定表示になる回帰でも通る。
   - 上記3点を明示的に assert し、両フックの状態表示を固定すること。

## 独立検証

- main 差分: 4ファイル、41 insertions / 19 deletions。すべて `tools/agent-dashboard` 配下で、範囲外変更なし。
- dashboard 全テスト: PASS。
- 変更4ファイルの ESLint・構文検査: PASS。
- `git diff --check main...HEAD -- tools/agent-dashboard`: PASS。
- リポジトリ全体 ESLint: FAIL。ただし差分外4ファイルに既存エラー4件で、今回の判定理由には含めない。
- needs の codd-gate command、`failureSummary`、`why`、`detail`、`blocked`、`decided=false` は保持され、実画面相当テストで検証失敗見出し・要約・context・判断材料の折り畳みも維持される。

## agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---:|---:|---:|---:|
| functional | Request Changes | 0 | 1 | 0 |
| ai-antipattern | Request Changes | 0 | 1 | 0 |
| architecture | LGTM | 0 | 0 | 1 |
| test | Request Changes | 0 | 1 | 0 |

architecture の suggestion（overview 専用関数の配置）は範囲外リファクタであり、合否には含めない。

<!-- verdict-json -->
```json
{
  "skill": "agent-reviewer",
  "verdict": "REQUEST_CHANGES",
  "blocking": true,
  "perspectives_executed": ["functional", "ai-antipattern", "architecture", "test"],
  "aggregated_blocking_issues": [
    {
      "from_perspective": "functional, ai-antipattern, test",
      "severity": "Warning",
      "summary": "別コマンドが設定済みの未結線状態で有効化手順と設定編集導線が消える",
      "location": "tools/agent-dashboard/src/renderer/renderer.js:1232-1254"
    },
    {
      "from_perspective": "test",
      "severity": "Warning",
      "summary": "両キー設定済み・両方未結線ケースで intake_cmd と両キーの設定状態を固定していない",
      "location": "tools/agent-dashboard/test/consistency-gate-ui.test.js:139"
    }
  ]
}
```
<!-- verdict-json -->

```json
{"ok": false, "issues": ["tools/agent-dashboard/src/renderer/renderer.js:1232-1254: 両キーに別コマンドが設定済みだと、codd-gate 未結線でも有効化手順と設定編集導線が消える", "tools/agent-dashboard/test/consistency-gate-ui.test.js:139-155 / test/overview-ui.test.js:345-353: intake_cmd と両キーの設定状態を固定していない"]}
```

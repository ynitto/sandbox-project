# verify report — t5〜t8 統合差分

verify=fail

## agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---|---:|---:|---:|
| functional | Request Changes | 0 | 2 | 0 |
| test | Request Changes | 0 | 2 | 0 |
| document | Request Changes | 0 | 4 | 0 |

総合判定: Request Changes。以下は重複を統合し、再作業可能な指摘だけに絞った。

## issues

1. `tools/agent-dashboard/package.json:10`、`src/features/agent-project/main/project.js:1435`、`src/features/agent-project/main/toolconfig.js:29`、`src/renderer/sections/needs.js:1794`、`test/detail-tabs-ui.test.js:452`、`test/overview-ui.test.js:304`: HEAD `dbd74f921` は最新 main `652184e15` を取り込めておらず、`git merge-tree --write-tree --messages HEAD main` が exit 1、6ファイルで content conflict になる。最新 main を統合し、package.json では main 側の全テストと consistency-gate 系3テストを併存、project.js では最新の state-repo/bus/liveness 変更と `consistencyGate` を併存、toolconfig.js では最新の共通 YAML parser を保持、needs/test harness では双方の引数・表示契約を保持して解消し、統合後に `npm test` と `git diff --check` を再実行すること。

2. `tools/agent-dashboard/src/features/agent-project/main/toolconfig.js:29-57` / `test/consistency-gate.test.js:40`: 手書きの行指向 YAML parser が公式設定解釈と一致しない。再現例では、引用符内に `#` を含み、その後ろに canonical command が続く値を dashboard は未結線・公式 PyYAML は結線済みと判定し、二重引用値の escaped `\\n` を dashboard は結線済み・公式は未結線と判定した。最新 main の `base/main/yaml.js` を使う `parseFlatYaml` を採用し、公式 `codd_gate_wiring.py` と同じ入力を突き合わせる契約テスト（通常、別コマンド、引用内 `#`、escaped newline）を追加すること。

3. `tools/agent-dashboard/src/renderer/renderer.js:1110-1161` / `README.md:103-117`: 未結線だが既存の汎用 `regression_cmd` / `intake_cmd` がある場合にも「次の行を書く」と案内し、YAML では重複キー、JSON では既存キー喪失または不正 JSON を誘発する。configured-but-unwired は「既存行・既存プロパティを置換」と明示し、既存フックを失うことへの確認を出すこと。JSON は既存トップレベル object にプロパティをマージし `root` 等を保持する貼付可能な案内へ変更すること。`<root>` が実パスへ置換必須のプレースホルダーであることも明記すること。

4. `tools/agent-dashboard/README.md:99-101` / `src/features/agent-project/main/project.js:1657`: 「agent-project 本体と同じ探索順」「実効値」と断定しているが、本体が `--config /custom/path` で起動された場合を dashboard は取得できず、別設定の有無・結線状態を表示する。解決済み config path が公式 instance/status 契約から得られるまでは「dashboard の自動探索結果」と限定表示し、明示 config は確認不能と示すこと。公式契約拡張が必要なら `@followup agent-project` とし、この差分内で本体を変更しないこと。

5. `tools/agent-dashboard/src/renderer/sections/needs.js:1307-1317`: `needGateSource()` は command/本文に任意の `codd-gate` があれば `codd-gate doctor` や単なる言及までゲート検査として扱い、「一貫性ゲートが止めた」と誤表示し得る。少なくとも canonical な `codd-gate ... verify` を要求し、regression は構造化 `failure-phase: regression` と記録された failure-command を優先、legacy fallback も同じ回帰記録内の canonical verify を必要条件にするテストを追加すること。

## 独立検証結果

- `npm test`: exit 0（ブランチ単体）
- 元要求の grep + `needs-diagnosis.test.js` + `overview-ui.test.js`: exit 0
- `consistency-gate.test.js`: 13/13 pass
- `needs-gate-integration.test.js`: 8/8 pass
- `git diff --check main...HEAD`: pass
- 差分: 13 files, +1223/-12。すべて `tools/agent-dashboard/**` 配下
- current main との merge-tree: exit 1、content conflict 6件

<!-- verdict-json -->
```json
{
  "skill": "agent-reviewer",
  "verdict": "REQUEST_CHANGES",
  "blocking": true,
  "perspectives_executed": ["functional", "test", "document"],
  "perspective_results": [
    {"perspective": "functional", "verdict": "REQUEST_CHANGES", "blocking": true, "severity_summary": {"critical": 0, "warning": 2, "suggestion": 0}},
    {"perspective": "test", "verdict": "REQUEST_CHANGES", "blocking": true, "severity_summary": {"critical": 0, "warning": 2, "suggestion": 0}},
    {"perspective": "document", "verdict": "REQUEST_CHANGES", "blocking": false, "severity_summary": {"critical": 0, "warning": 4, "suggestion": 0}}
  ],
  "aggregated_blocking_issues": [
    {"from_perspective": "functional/test", "severity": "Warning", "summary": "最新 main と6ファイルで content conflict が発生する", "location": "tools/agent-dashboard/package.json:10 ほか"},
    {"from_perspective": "test", "severity": "Warning", "summary": "YAML境界入力で公式結線判定と dashboard 判定が逆転する", "location": "tools/agent-dashboard/src/features/agent-project/main/toolconfig.js:29"},
    {"from_perspective": "functional/document", "severity": "Warning", "summary": "既存設定を壊し得る有効化案内になっている", "location": "tools/agent-dashboard/src/renderer/renderer.js:1110"},
    {"from_perspective": "document", "severity": "Warning", "summary": "明示 --config を読めないのに実効設定と断定する", "location": "tools/agent-dashboard/src/features/agent-project/main/project.js:1657"},
    {"from_perspective": "document", "severity": "Warning", "summary": "任意の codd-gate 文字列をゲート失敗と誤分類し得る", "location": "tools/agent-dashboard/src/renderer/sections/needs.js:1307"}
  ]
}
```
<!-- verdict-json -->

{"ok": false, "issues": ["最新 main と6ファイルで content conflict", "公式 YAML 解釈と結線判定が不一致", "既存設定を壊し得る有効化導線", "--config 明示時の実効設定を誤表示", "needs の codd-gate 由来判定が過広"]}

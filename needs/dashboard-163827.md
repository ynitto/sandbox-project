---
status: proposed
date: 2026-08-01
decision-makers: [human]
task-id: dashboard-163827
kind: review
risk: med
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/README.md","tools/agent-dashboard/package.json","tools/agent-dashboard/src/features/agent-project/main/project.js","tools/agent-dashboard/src/features/agent-project/main/toolconfig.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/src/renderer/sections/needs.js","tools/agent-dashboard/src/renderer/sections/overview.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/consistency-gate.test.js","tools/agent-dashboard/test/detail-tabs-ui.test.js","tools/agent-dashboard/test/needs-diagnosis.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js","tools/agent-dashboard/test/overview-ui.test.js"],"files_total":13,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
verification: {"criteria":[{"id":1,"text":"固定検証コマンド: `grep -nE 'regression_cmd|intake_cmd|一貫性ゲート' tools/agent-dashboard/src/renderer/renderer.js tools/agent-dashboard/src/features/agent-project/main/project.js && n`","verdict":"pass","evidence":{"commands":["grep -nE 'regression_cmd|intake_cmd|一貫性ゲート' tools/agent-dashboard/src/renderer/renderer.js tools/agent-dashboard/src/features/agent-project/main/project.js && node tools/agent-dashboard/test/needs-diagnosis.test.js && node tools/agent-dashboard/test/overview-ui.test.js"],"output":"- 検証コマンドが対象を見つけられない失敗を要約する\nok - 見つからない相対パスはツールに依存せず実行条件と対処を提示する\nok - 連鎖の途中で沈黙した工程は「失敗した工程」として名指しされる\nok - 旧形式（工程の記録なし）でも「テストの失敗ではない」ことは言う\nok - テストの失敗件数を要約する\nok - コマンド不在を要約する\nok - 解釈できない失敗は終了コードだけ添える\nok - 手掛かりが無ければ要約しない（生の情報を隠さない）\nok - 差分を成果物と内部の実行記録に分ける\nok - 差分が内部の実行記録だけなら「痩せた判断材料」として印を付ける\nok - 差分リストは次のセクションで終わる（検証行を取り込まない）\nok - 一貫性ゲートの回帰失敗は原因を読め、done にしない\n\n12 passed\noverview-ui: all tests passed","files":[]},"note":""}],"report":"verifications/dashboard-163827/635bc8d17c339fa6d29a9f2efd9934bc09ab8d42.md","pass":1}
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 検証は通っている（verify=PASS）。人の検収を待っている理由: このタスクが承認ゲートの対象（review / policy.gate）。内容が良ければ approve で done 確定、直したいことがあれば下に書いて差し戻す
- 状態: review（検収待ち・verify=PASS）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/dashboard-163827`（13 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827`
- 変更ファイル（13 件）:
    - tools/agent-dashboard/README.md
    - tools/agent-dashboard/package.json
    - tools/agent-dashboard/src/features/agent-project/main/project.js
    - tools/agent-dashboard/src/features/agent-project/main/toolconfig.js
    - tools/agent-dashboard/src/renderer/renderer.js
    - tools/agent-dashboard/src/renderer/sections/needs.js
    - tools/agent-dashboard/src/renderer/sections/overview.js
    - tools/agent-dashboard/test/consistency-gate-ui.test.js
    - tools/agent-dashboard/test/consistency-gate.test.js
    - tools/agent-dashboard/test/detail-tabs-ui.test.js
    - tools/agent-dashboard/test/needs-diagnosis.test.js
    - tools/agent-dashboard/test/needs-gate-integration.test.js
    - …他 1 件
- 実行先: local
- 到達工程: verify（検証）
- 検証: `grep -nE 'regression_cmd|intake_cmd|一貫性ゲート' tools/agent-dashboard/src/renderer/renderer.js tools/agent-dashboard/src/features/agent-project/main/project.js && node tools/agent-dashboard/test/needs-diagnosis.test.js && node tools/agent-dashboard/test/overview-ui.test.js` → PASS（基準 1 件中 1 件 pass（agent-flow runner の receipt を検算して採用））

## リスク
- 総合: 中（protect/avoid=高、リトライ・大差分・合成 verify=中）
- リトライ: 9 回（NG 積み直しを経た成果）
- 変更ファイル: 13 件（tools/agent-dashboard/README.md, tools/agent-dashboard/package.json, tools/agent-dashboard/src/features/agent-project/main/project.js, tools/agent-dashboard/src/features/agent-project/main/toolconfig.js, tools/agent-dashboard/src/renderer/renderer.js 他 8 件）
- 投入時採点: c=2 r=1 a=1（c=複雑さ r=リスク a=曖昧さ・各1-3）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve dashboard-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

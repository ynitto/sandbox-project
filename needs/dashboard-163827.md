---
status: proposed
date: 2026-07-26
decision-makers: [human]
task-id: dashboard-163827
kind: review
risk: med
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/package.json","tools/agent-dashboard/src/features/agent-project/main/project.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/src/renderer/sections/needs.js","tools/agent-dashboard/src/renderer/sections/overview.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/consistency-gate.test.js","tools/agent-dashboard/test/detail-tabs-ui.test.js","tools/agent-dashboard/test/needs-diagnosis.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js","tools/agent-dashboard/test/overview-ui.test.js"],"files_total":11,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 検証は通っている（verify=PASS）。人の検収を待っている理由: このタスクが承認ゲートの対象（review / policy.gate）。内容が良ければ approve で done 確定、直したいことがあれば下に書いて差し戻す
- 状態: review（検収待ち・verify=PASS）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/dashboard-163827`（11 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827`
- 変更ファイル（11 件）:
    - tools/agent-dashboard/package.json
    - tools/agent-dashboard/src/features/agent-project/main/project.js
    - tools/agent-dashboard/src/renderer/renderer.js
    - tools/agent-dashboard/src/renderer/sections/needs.js
    - tools/agent-dashboard/src/renderer/sections/overview.js
    - tools/agent-dashboard/test/consistency-gate-ui.test.js
    - tools/agent-dashboard/test/consistency-gate.test.js
    - tools/agent-dashboard/test/detail-tabs-ui.test.js
    - tools/agent-dashboard/test/needs-diagnosis.test.js
    - tools/agent-dashboard/test/needs-gate-integration.test.js
    - tools/agent-dashboard/test/overview-ui.test.js
- 実行先: local
- 到達工程: verify（検証）
- 検証: `grep -nE 'regression_cmd|intake_cmd|一貫性ゲート' tools/agent-dashboard/src/renderer/renderer.js tools/agent-dashboard/src/features/agent-project/main/project.js && node tools/agent-dashboard/test/needs-diagnosis.test.js && node tools/agent-dashboard/test/overview-ui.test.js` → PASS（exit=0 途中で沈黙した工程は「失敗した工程」として名指しされる ok - 旧形式（工程の記録なし）でも「テストの失敗ではない」ことは言う ok - テストの失敗件数を要約する ok - コマンド不在を要約する ok - 解釈できない失敗は終了コードだけ添える ok - 手掛かりが無ければ要約しない（生の情報を隠さない） ok - 差分を成果物と内部の実行記録に分ける ok - 差分が内部の実）

## リスク
- 総合: 中（protect/avoid=高、リトライ・大差分・合成 verify=中）
- 変更ファイル: 11 件（tools/agent-dashboard/package.json, tools/agent-dashboard/src/features/agent-project/main/project.js, tools/agent-dashboard/src/renderer/renderer.js, tools/agent-dashboard/src/renderer/sections/needs.js, tools/agent-dashboard/src/renderer/sections/overview.js 他 6 件）
- 投入時採点: c=2 r=1 a=1（c=複雑さ r=リスク a=曖昧さ・各1-3）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve dashboard-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

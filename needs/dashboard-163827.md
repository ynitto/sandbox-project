---
status: proposed
date: 2026-08-01
decision-makers: [human]
task-id: dashboard-163827
kind: blocked
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/README.md","tools/agent-dashboard/package.json","tools/agent-dashboard/src/features/agent-project/main/project.js","tools/agent-dashboard/src/features/agent-project/main/toolconfig.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/src/renderer/sections/needs.js","tools/agent-dashboard/src/renderer/sections/overview.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/consistency-gate.test.js","tools/agent-dashboard/test/detail-tabs-ui.test.js","tools/agent-dashboard/test/needs-diagnosis.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js","tools/agent-dashboard/test/overview-ui.test.js"],"files_total":13,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 繰り返し NG（retries=11）: 基準 2 件中 1 件 pass: [fail] 最新 target `main` が成果 revision に統合済み — 最新 target が成果 revision の祖先ではありません（agent-flow runner の receipt を検算して採用）
- 状態: blocked（agent-project の判断待ち）

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
- 検証: `grep -nE 'regression_cmd|intake_cmd|一貫性ゲート' tools/agent-dashboard/src/renderer/renderer.js tools/agent-dashboard/src/features/agent-project/main/project.js && node tools/agent-dashboard/test/needs-diagnosis.test.js && node tools/agent-dashboard/test/overview-ui.test.js` → FAIL（基準 2 件中 1 件 pass: [fail] 最新 target `main` が成果 revision に統合済み — 最新 target が成果 revision の祖先ではありません（agent-flow runner の receipt を検算して採用））

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 上の [ ] を [x] にした時だけ反映されます（書きかけでの誤発火を防ぐため）。
     下に修正方針・指示を書いてください。空のままでも [x] なら『そのまま再実行』。
     コマンドなら `agent-project approve dashboard-163827`。 -->

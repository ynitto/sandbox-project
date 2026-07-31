---
status: proposed
date: 2026-08-01
decision-makers: [human]
task-id: dashboard-163827
kind: review
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/README.md","tools/agent-dashboard/package.json","tools/agent-dashboard/src/features/agent-project/main/project.js","tools/agent-dashboard/src/features/agent-project/main/toolconfig.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/src/renderer/sections/needs.js","tools/agent-dashboard/src/renderer/sections/overview.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/consistency-gate.test.js","tools/agent-dashboard/test/detail-tabs-ui.test.js","tools/agent-dashboard/test/needs-diagnosis.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js","tools/agent-dashboard/test/overview-ui.test.js"],"files_total":13,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 承認されたが成果ブランチを統合できない: main と ap/dashboard-163827 の自動統合で競合しました。成果ブランチを更新して再検収してください: Auto-merging tools/agent-dashboard/README.md
Auto-merging tools/agent-dashboard/package.json
CONFLICT (content): Merge conflict in tools/agent-dashboard/package.json
Auto-merging tools/agent-dashboard/src/features/agent-project/main/project.js
CONFLICT (content): Merge conflict in tools/agent-dashboard/src/features/agent-project/main/project.js
Auto-merging tools/agent-dashboard/src/features/agent-project/main/toolconfig.js
CONFLICT (content): Merge conflict in tools/agent-dashboard/src/features
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

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve dashboard-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

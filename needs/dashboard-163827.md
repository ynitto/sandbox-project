---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: dashboard-163827
kind: review
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md","tools/agent-dashboard/README.md","tools/agent-dashboard/src/features/cowork/main/cowork.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js"],"files_total":6,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
failure-class: integration
failure-chain: integration
failure-phase: verify
verify-verdict: failed
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 検証後に target main が更新されました（9c196643e1a8 → 2f74ce9be0c2）。最新 target を統合して再検証してください
- 状態: review（検収待ち・verify=PASS）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve dashboard-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

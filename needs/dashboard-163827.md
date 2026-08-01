---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: dashboard-163827
kind: review
risk: med
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/dashboard-163827","ref":"origin/ap/dashboard-163827","files":["tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md","tools/agent-dashboard/README.md","tools/agent-dashboard/src/features/cowork/main/cowork.js","tools/agent-dashboard/src/renderer/renderer.js","tools/agent-dashboard/test/consistency-gate-ui.test.js","tools/agent-dashboard/test/needs-gate-integration.test.js"],"files_total":6,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827","mr_url":""}]
verification: {"criteria":[{"id":1,"text":"最新 target `main` が成果 revision に統合済み","verdict":"pass","evidence":{"commands":[],"output":"9c196643e1a8e7a3efd3e47cc36f535f782d438a","files":[]},"note":""},{"id":2,"text":"固定検証コマンド: `echo \"done\"`","verdict":"pass","evidence":{"commands":["echo \"done\""],"output":"done","files":[]},"note":""}],"report":"verifications/dashboard-163827/016a4bde9bf90b57d6cdc35571fcb17674079ab9.md","pass":2}
---

# 要対応: dashboard-163827 — dashboard で一貫性ゲートの状態把握と有効化を支援する

## Context and Problem Statement

- なぜ: 検証は通っている（verify=PASS）。人の検収を待っている理由: このタスクが承認ゲートの対象（review / policy.gate）。内容が良ければ approve で done 確定、直したいことがあれば下に書いて差し戻す
- 状態: review（検収待ち・verify=PASS）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/dashboard-163827`（6 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/dashboard-163827`
- 変更ファイル（6 件）:
    - tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md
    - tools/agent-dashboard/README.md
    - tools/agent-dashboard/src/features/cowork/main/cowork.js
    - tools/agent-dashboard/src/renderer/renderer.js
    - tools/agent-dashboard/test/consistency-gate-ui.test.js
    - tools/agent-dashboard/test/needs-gate-integration.test.js
- 実行先: local
- 到達工程: verify（検証）
- 検証: `echo "done"` → PASS（基準 2 件中 2 件 pass（agent-flow runner の receipt を検算して採用））

## リスク
- 総合: 中（protect/avoid=高、リトライ・大差分・合成 verify=中）
- リトライ: 18 回（NG 積み直しを経た成果）
- 変更ファイル: 6 件（tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md, tools/agent-dashboard/README.md, tools/agent-dashboard/src/features/cowork/main/cowork.js, tools/agent-dashboard/src/renderer/renderer.js, tools/agent-dashboard/test/consistency-gate-ui.test.js 他 1 件）
- 投入時採点: c=2 r=1 a=1（c=複雑さ r=リスク a=曖昧さ・各1-3）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve dashboard-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

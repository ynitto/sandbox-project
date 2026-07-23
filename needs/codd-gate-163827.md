---
status: proposed
date: 2026-07-24
decision-makers: [human]
task-id: codd-gate-163827
kind: review
risk: med
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/codd-gate-163827","ref":"origin/ap/codd-gate-163827","files":["docs/designs/codd-gate-design.md","tools/agent-project/README.md"],"files_total":2,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/codd-gate-163827","mr_url":""}]
---

# 要対応: codd-gate-163827 — codd-gate 連携の目標境界を設計書に固定する

## Context and Problem Statement

- なぜ: 検証は通っている（verify=PASS）。人の検収を待っている理由: このタスクが承認ゲートの対象（review / policy.gate）。内容が良ければ approve で done 確定、直したいことがあれば下に書いて差し戻す
- 状態: review（検収待ち・verify=PASS）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/codd-gate-163827`（2 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/codd-gate-163827`
- 変更ファイル（2 件）:
    - docs/designs/codd-gate-design.md
    - tools/agent-project/README.md
- 実行先: local
- 到達工程: verify（検証）
- 検証: `grep -nE 'agent_project.*(import|結合|依存).*(しない|外|禁止)|パッケージ.*(codd_gate|sibling)|有効化は設定' tools/agent-project/README.md && grep -nE 'regression_cmd|intake_cmd|codd_gate_\*\.py|自動検出' tools/agent-project/README.md && test -f docs/designs/codd-gate-design.md && grep -nE 'agent_project パッケージ|_apply_codd_gate|sibling|汎用フック' docs/designs/codd-gate-design.md` → PASS（exit=0 _auto_wiring`）はパッケージ内に 367:しか現れず、sibling の `codd_gate_*.py` や tests には出ない（それらは `resolve_codd_gate` などを正当に持つ）。 377:パスを `tools/agent-project` 全体へ広げると、sibling の `resolve_codd_gate` や tests の `impo）

## リスク
- 総合: 中（protect/avoid=高、リトライ・大差分・合成 verify=中）
- リトライ: 1 回（NG 積み直しを経た成果）
- 変更ファイル: 2 件（docs/designs/codd-gate-design.md, tools/agent-project/README.md）
- 投入時採点: c=2 r=1 a=1（c=複雑さ r=リスク a=曖昧さ・各1-3）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve codd-gate-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

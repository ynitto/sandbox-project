---
status: proposed
date: 2026-07-26
decision-makers: [human]
task-id: agent_project-codd_gate-163827
kind: review
risk: med
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/agent_project-codd_gate-163827","ref":"origin/ap/agent_project-codd_gate-163827","files":["tools/agent-project/agent_project/configfile.py","tools/agent-project/agent_project/model.py","tools/agent-project/tests/test_agent_project.py","tools/agent-project/tests/test_autonomy.py","tools/agent-project/tests/test_backlog.py","tools/agent-project/tests/test_codd_gate_wiring.py","tools/agent-project/tests/test_config.py","tools/agent-project/tests/test_doctor.py"],"files_total":8,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/agent_project-codd_gate-163827","mr_url":""}]
---

# 要対応: agent_project-codd_gate-163827 — agent_project を codd_gate 非依存の汎用フックへ整理する

## Context and Problem Statement

- なぜ: 検証は通っている（verify=PASS）。人の検収を待っている理由: このタスクが承認ゲートの対象（review / policy.gate）。内容が良ければ approve で done 確定、直したいことがあれば下に書いて差し戻す
- 状態: review（検収待ち・verify=PASS）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/agent_project-codd_gate-163827`（8 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/agent_project-codd_gate-163827`
- 変更ファイル（8 件）:
    - tools/agent-project/agent_project/configfile.py
    - tools/agent-project/agent_project/model.py
    - tools/agent-project/tests/test_agent_project.py
    - tools/agent-project/tests/test_autonomy.py
    - tools/agent-project/tests/test_backlog.py
    - tools/agent-project/tests/test_codd_gate_wiring.py
    - tools/agent-project/tests/test_config.py
    - tools/agent-project/tests/test_doctor.py
- 実行先: local
- 到達工程: verify（検証）
- 検証: `PYTHONPATH=tools/agent-project python3 tools/agent-project/tests/test_agent_project.py TestIntake.test_run_intake_enqueues_and_dedups_by_id TestLoopEngineering.test_regression_gate_blocks_on_failure TestLoopEngineering.test_regression_gate_passes && ! git grep -n -E '(^|[[:space:]])(import|from)[[:space:]]+codd_gate|_apply_codd_gate|_codd_gate' -- tools/agent-project/agent_project` → PASS（exit=0 --- 通知（要対応）--- # 要対応（agent-project）  ## 判断待ち（blocked） - T1: x     なぜ: 回帰検知: グローバル検査 `external-regression-hook` 失敗 — hook failed     対応: needs/T1.md に方針を書く、または `approve T1` / `hold T1`  ... ----）

## リスク
- 総合: 中（protect/avoid=高、リトライ・大差分・合成 verify=中）
- リトライ: 1 回（NG 積み直しを経た成果）
- 変更ファイル: 8 件（tools/agent-project/agent_project/configfile.py, tools/agent-project/agent_project/model.py, tools/agent-project/tests/test_agent_project.py, tools/agent-project/tests/test_autonomy.py, tools/agent-project/tests/test_backlog.py 他 3 件）
- 投入時採点: c=2 r=2 a=1（c=複雑さ r=リスク a=曖昧さ・各1-3）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して done 確定するなら `agent-project approve agent_project-codd_gate-163827`。
     差し戻すなら下に修正方針を書いて [x] にする（再実行されます）。 -->

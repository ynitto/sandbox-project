---
status: proposed
date: 2026-08-03
decision-makers: [human]
task-id: document-msbqisx2-5
kind: blocked
delivery: [{"name":"sandbox","role":"write","url":"https://github.com/ynitto/sandbox","path":"/Users/nitto/Workspace/sandbox","base":"main","target":"main","branch":"ap/document-msbqisx2-5","ref":"origin/ap/document-msbqisx2-5","files":["tools/agent-project/README.md","tools/agent-project/agent_project/_head.py","tools/agent-project/agent_project/brief.py","tools/agent-project/agent_project/decisions.py","tools/agent-project/agent_project/stategit.py","tools/agent-project/tests/_privacy_fixture.py","tools/agent-project/tests/test_privacy.py"],"files_total":7,"diff_cmd":"git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/document-msbqisx2-5","mr_url":""}]
---

# 要対応: document-msbqisx2-5 — 共有禁止項目（redaction）を契約テスト化する

## Context and Problem Statement

- なぜ: verify 未定義（工程は完了しています。完了条件が無いため自動では done にできません。成果を確認し、問題なければ approve してください）
- 状態: blocked（agent-project の判断待ち）

## 判断材料（成果物の所在・差分・検証）
- 成果物: ブランチ `ap/document-msbqisx2-5`（7 ファイル変更・base `main`）
- 所在: /Users/nitto/Workspace/sandbox
- 差分を見る: `git -C /Users/nitto/Workspace/sandbox diff main...origin/ap/document-msbqisx2-5`
- 変更ファイル（7 件）:
    - tools/agent-project/README.md
    - tools/agent-project/agent_project/_head.py
    - tools/agent-project/agent_project/brief.py
    - tools/agent-project/agent_project/decisions.py
    - tools/agent-project/agent_project/stategit.py
    - tools/agent-project/tests/_privacy_fixture.py
    - tools/agent-project/tests/test_privacy.py
- 実行先: local
- 到達工程: act（実装）
- 検証: 未定義（自動では完了にできないため、成果を確認して承認してください）

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 上の [ ] を [x] にした時だけ反映されます（書きかけでの誤発火を防ぐため）。
     下に修正方針・指示を書いてください。空のままでも [x] なら『そのまま再実行』。
     コマンドなら `agent-project approve document-msbqisx2-5`。 -->

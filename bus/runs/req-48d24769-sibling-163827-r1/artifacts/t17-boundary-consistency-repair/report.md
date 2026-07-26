# t17 境界整合性修復

## 成果

- `codd_gate_wiring.py` から `upsert_yaml_text()` と `apply_yaml_file()` を削除した。診断 CLI は読み取り専用のまま残した。
- `codd_gate_regression.py` に `regression_cmd` の冪等更新を一元化した。生成 CLI は `detect_status()` で実体だけを検出し、未検出時は既存 YAML を変更しない。
- 永続化境界と未検出時の no-op をテストで固定した。
- README、GUIDE、設計 §4.1 を更新した。結線診断の唯一の正準入口は `python3 tools/agent-project/codd_gate_wiring.py --config ...` とし、package doctor 経路を除いた。
- 文書は `slop-police` の基準で見直し、実装にない version、schema、capability の保証を削った。
- `agent_project/` と dashboard は変更していない。

## 検証

- `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`
  - 111 tests、成功。
- README の `codd_gate_regression` / `regression_cmd` / `intake_cmd` 完了条件 grep
  - 成功。
- README の旧自動配線記述がないことの否定 grep
  - 成功。
- `agent_project/` に codd-gate の再結合がないことの否定 grep
  - 成功。
- `git diff --quiet -- tools/agent-project/agent_project tools/agent-dashboard tools/agent-project/dashboard`
  - 成功。禁止範囲の差分なし。
- `git diff --check`
  - 成功。

## 前提・未解決事項・範囲外

- 「変更してよいのは tools/agent-project 配下のみ」と、`docs/designs/codd-gate-design.md` §4.1 の更新指示が競合していた。3指摘の完了に必須な明示対象なので、この設計書だけを例外として更新した。
- 依存成果の指摘は、現在の作業ブランチに残る不整合への修正条件として採用した。
- 未解決事項はない。範囲外の問題も見つからなかった。

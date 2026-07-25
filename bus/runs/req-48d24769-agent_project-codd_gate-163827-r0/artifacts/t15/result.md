verify=fail

- 指定コマンド
  `PYTHONPATH=tools/agent-project python3 tools/agent-project/tests/test_agent_project.py TestLoopEngineering.test_regression_gate_passes`
  は、`tools/agent-project/tests/test_agent_project.py` が存在しないため exit 2 で失敗した。
- 同名テストは `tools/agent-project/tests/test_autonomy.py:513` に存在し、実在パスへ置き換えた同等コマンドは exit 0 で成功した。
- `main...HEAD` の差分は7ファイルで、すべて `tools/agent-project` 配下。未コミット差分なし、`git diff --check` 成功。

{"ok": false, "issues": ["tools/agent-project/tests/test_agent_project.py が存在せず、契約された regression gate コマンドが exit 2 になる。verify コマンドを tools/agent-project/tests/test_autonomy.py TestLoopEngineering.test_regression_gate_passes に更新するか、契約どおり旧パスで実行可能なテスト入口を tools/agent-project/tests/test_agent_project.py に復元すること。"]}

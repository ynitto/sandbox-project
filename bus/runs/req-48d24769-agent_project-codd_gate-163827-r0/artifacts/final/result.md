verify=pass

- 完全受入コマンドを一列で再実行し、終了コード 0。
  - intake: 1 test / OK
  - regression failure: 1 test / OK
  - regression pass: 1 test / OK
  - 否定 `git grep`: 一致なし
- 互換入口の直接実行: 18 tests / OK。
- `test_agent_project.py` の discovery: 0 tests / OK（分割先の重複収集なし）。
- `git diff --check main...HEAD`: 成功。
- worktree: clean。
- `main...HEAD`: 8ファイルすべて `tools/agent-project` 配下。

{"ok": true, "issues": []}

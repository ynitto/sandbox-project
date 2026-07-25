# 成果

`tools/agent-project/tests/test_agent_project.py` を互換入口として追加した。
分割先の `tests.test_backlog.TestIntake` と
`tests.test_autonomy.TestLoopEngineering` を再公開し、直接実行時は
`unittest.main()` を呼ぶ。実装本体・mock・禁止識別子には変更を加えていない。

## 検証

以下を同じ作業ディレクトリで再実行し、すべて終了コード 0。

- `PYTHONPATH=tools/agent-project python3 tools/agent-project/tests/test_agent_project.py TestIntake.test_run_intake_enqueues_and_dedups_by_id`
  - `Ran 1 test` / `OK`
- `PYTHONPATH=tools/agent-project python3 tools/agent-project/tests/test_agent_project.py TestLoopEngineering.test_regression_gate_blocks_on_failure`
  - `Ran 1 test` / `OK`
- `PYTHONPATH=tools/agent-project python3 tools/agent-project/tests/test_agent_project.py TestLoopEngineering.test_regression_gate_passes`
  - `Ran 1 test` / `OK`
- `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  - 一致なし。`git grep` の exit 1 を否定して終了コード 0。
- `git diff --check`
  - 終了コード 0。
- `git status --short`
  - 今回の変更は未追跡の `tools/agent-project/tests/test_agent_project.py` 1ファイルのみ。

## 前提・未解決事項・範囲外

- judge の採用結果どおり、失敗原因は固定コマンドが参照する互換入口の欠落だけであり、
  分割先テスト・ソース・mock・禁止識別子には修正対象がないとした。
- 「指定3テスト」は judge/t13/t15 と分割先の実測から、intake の冪等テスト、
  regression gate の failure/pass 各テストと解釈した。
- 未解決事項および範囲外で新たに見つけた問題はない。

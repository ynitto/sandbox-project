# t4 成果報告

## 成果

- `agent_project/configfile.py` の `intake_cmd` 既定コメントと CLI help を、特定 provider の名称・例示に依存しない「JSON を返す外部の決定的検出器」契約へ変更した。
- 旧 private 関数名の不在を直接検査していた `tests/test_config.py` を、設定ファイルの `hooks` が `resolve_config` → `build_config` → `Config.hooks` の既存経路を通って保持される振る舞いテストへ置換した。
- `repos.json` の有無で `regression_cmd` / `intake_cmd` を自動補完しない既存3ケースは維持した。
- 変更ファイルは `tools/agent-project/agent_project/configfile.py` と `tools/agent-project/tests/test_config.py` のみ。commit / push / branch 操作は行っていない。

## 検証

- `python3 -m unittest tests.test_config`: 53件成功。
- `python3 -m unittest tests.test_backlog tests.test_config tests.test_doctor tests.test_autonomy tests.test_codd_gate_wiring tests.test_codd_gate_regression`: 241件成功。
- `python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`: 111件成功。
- `git diff --check`: 問題なし。
- `configfile.py` に対する `codd[_-]gate|_apply_codd_gate` grep: 0件。

## 前提・未解決事項・範囲外

- 前提: 現行 worktree では `_apply_codd_gate_auto_wiring` は既に削除済みで、確定仕様の汎用自動配線は `agent_project/hooks.py::_hook_provider` に実装済み（明示 `hooks` 優先、未指定時 sibling 能力探索）だった。そのため configfile に重複する配線関数を新設せず、設定供給経路と観測可能な回帰契約を確定仕様へ揃えることを本タスクの完了条件とした。
- 未解決事項なし。
- 範囲外: `agent_project` の他ファイルに残る provider 固有文言は、依存成果のファイル別変更表では別担当箇所のため変更していない。

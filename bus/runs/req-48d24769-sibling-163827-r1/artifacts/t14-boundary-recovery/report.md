# t14 成果報告

## 成果・サマリー

- `codd_gate_status.py` は既存 API と挙動を維持し、パッケージ外 sibling として CLI の検出結果・
  finding・no-op 状態だけを扱う説明へ更新した。`agent_project` への自動配線、設定書き込み、
  package doctor 登録は非責務と明記した。
- `codd_gate_regression.py` は `codd_gate_routing` の推奨文字列生成を再利用し、成功時 JSON に
  `regression_cmd` と `intake_cmd` を出すようにした。既存の `cmd` フィールドは後方互換のため維持した。
- YAML へ冪等注入する対象は従来どおり `regression_cmd` だけで、`intake_cmd` は案内のみ。
  build_config のメモリ自動配線や `agent_project` パッケージ内への再結合は行っていない。
- 変更は `tools/agent-project/codd_gate_status.py`、`codd_gate_regression.py`、
  `tests/test_codd_gate_regression.py` の3ファイルだけ。

## 検証内容と結果

- `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`
  - 115 tests、すべて成功。
- `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests`
  - 終了コード 0。
- `python3 -m py_compile tools/agent-project/codd_gate_status.py tools/agent-project/codd_gate_regression.py`
  - 成功。
- `python3 tools/agent-project/codd_gate_regression.py --help`
  - 成功。
- README の完了条件 grep と禁止語句の不在確認
  - 成功。
- `git diff --check`
  - 成功。

## 採用した前提・未解決事項・範囲外

- 「必要な CLI 所見」は、status の既存 finding と regression CLI の機械可読な推奨値を指すと解釈した。
  package doctor へ自動登録する意味には取っていない。
- t7 の「regression_cmd・intake_cmd の案内を一貫させる」は、両方を同じ routing 正本から生成して
  JSON に出すことと解釈した。既存境界に従い `intake_cmd` は自動注入しない。
- t2/t7 の元変更は rebase 競合で失敗していたため、成果報告と失敗コミットを現 HEAD に照合し、
  README の重複変更を除いた最小差分だけを回収した。
- 未解決事項なし。範囲外の `agent_project` パッケージ、dashboard、設計書には変更を加えていない。

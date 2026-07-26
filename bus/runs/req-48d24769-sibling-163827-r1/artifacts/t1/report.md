# t1 成果報告

## 成果・サマリー

- `tools/agent-project/codd_gate_detect.py` のモジュール説明を新境界へ更新した。
- 責務を、パッケージ外 sibling としての codd-gate 実体解決、および推奨設定の可否判定に必要な
  バージョン・repos schema・サブコマンド能力の生検出値に限定した。
- 利用可否判定、推奨コマンド生成、YAML 注入、doctor 所見、`agent_project` パッケージへの結線は
  対象外であることを明記した。
- 公開 API は `codd_gate_wiring.py` の既存呼び出し契約に必要なため変更していない。

変更ファイル:

- `tools/agent-project/codd_gate_detect.py`

## 検証内容と結果

- `python3 -m unittest tools/agent-project/tests/test_codd_gate_detect.py`
  - 23件、全件 PASS。
- `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`
  - 115件、全件 PASS。
- `git diff --check`
  - PASS。
- `git status --short`
  - 指定ファイル `tools/agent-project/codd_gate_detect.py` のみ変更。

## 採用した前提・未解決事項・範囲外

- 完了条件は、検出 API を削ることではなく、同 API がパッケージ外 sibling の生検出値だけを提供し、
  後段の推奨判定に必要な情報を維持すること、と解釈した。
- `get_version`、`check_repos_schema_compat`、`detect_capabilities` は
  `codd_gate_wiring.py` が推奨可否を判断する既存契約なので維持した。
- 未解決事項なし。
- `agent_project` パッケージ内と agent-dashboard は確認・変更していない。

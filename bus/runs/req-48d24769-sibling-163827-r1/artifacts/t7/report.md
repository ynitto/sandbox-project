# t7 成果報告

## 成果・サマリー

- `codd_gate_regression.py` の `--config` を必須化し、設定ファイルを明示しない実行では
  argparse の使用法エラー（終了コード 2）になるようにした。
- 明示実行例と通常の設定先を、現行の設定探索境界である
  `.agents/agent-project.yaml` に統一した。
- 成功時 JSON は従来どおり `regression_cmd` と `intake_cmd` の推奨値を同じ routing 正本から案内する。
  YAML へ冪等注入するのは `regression_cmd` のみで、`intake_cmd` は人または install 手順が設定する、
  という境界を CLI ヘルプ、README、GUIDE で一致させた。
- `agent_project` パッケージ内と dashboard は変更していない。

変更ファイル:

- `tools/agent-project/codd_gate_regression.py`
- `tools/agent-project/tests/test_codd_gate_regression.py`
- `tools/agent-project/README.md`
- `tools/agent-project/GUIDE.md`

## 検証内容と結果

- 対象テスト:
  `python3 -m unittest tools/agent-project/tests/test_codd_gate_regression.py tools/agent-project/tests/test_codd_gate_routing.py`
  — 34 件、全件 PASS。
- codd-gate 系テスト:
  `python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`
  — 106 件、全件 PASS。
- 全体テスト:
  `python3 -m unittest discover -s tools/agent-project/tests`
  — 956 件、全件 PASS。
- `python3 tools/agent-project/codd_gate_regression.py --help`
  — `--config` 必須、`.agents/agent-project.yaml` の例、両コマンドの案内境界を確認。
- `git diff --check` — PASS。
- 変更パス確認 — すべて `tools/agent-project` 配下。

## 採用した前提・未解決事項・範囲外

- 完了条件は、連携を暗黙に有効化せず、利用者が `--config` で対象 YAML を明示し、
  `regression_cmd` の注入値と `intake_cmd` の手動設定値を同じ実行結果から判断できること、
  さらに現行の `.agents/` 設定境界と利用手順が一致すること、と解釈した。
- `codd_gate_regression.py` の既存責務を維持し、`intake_cmd` の自動注入は追加していない。
- 未解決事項なし。
- 範囲外: `agent_project` パッケージ内への再結合、dashboard 変更は行っていない。
- @followup `codd_gate_wiring.py` 単体の docstring・省略時既定値には旧
  `.agent/agent-project.yaml` が残る。今回の README/GUIDE は `--config .agents/...` を明示するため
  本タスクの利用経路は壊れないが、wiring CLI 自体の既定境界を統一する別タスクで更新するとよい。

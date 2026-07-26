# t7 成果報告

## 成果・サマリー

- `codd_gate_regression.py --config` の実行例をリポジトリルート基準の
  `python3 tools/agent-project/codd_gate_regression.py --config ...` に統一した。
- 成功時 JSON に `regression_cmd` と `intake_cmd` の推奨値を併記した。既存利用者向けの
  `cmd` フィールドは `regression_cmd` の別名として維持した。
- YAML へ冪等注入する対象は従来どおり `regression_cmd` 1行だけとし、`intake_cmd` は案内のみで
  人または install 手順が設定する境界を CLI ヘルプと README に明記した。
- 推奨文字列の生成は `codd_gate_routing.recommend_regression_cmd` /
  `recommend_intake_cmd` を再利用し、案内と注入値のずれを防いだ。

変更ファイル:

- `tools/agent-project/codd_gate_regression.py`
- `tools/agent-project/tests/test_codd_gate_regression.py`
- `tools/agent-project/README.md`

## 検証内容と結果

- 対象テスト:
  `python3 -m unittest tools/agent-project/tests/test_codd_gate_regression.py tools/agent-project/tests/test_codd_gate_routing.py`
  — 43件、全件 PASS。
- 全体テスト:
  `python3 -m unittest discover -s tools/agent-project/tests`
  — 962件、全件 PASS。
- `python3 tools/agent-project/codd_gate_regression.py --help`
  — リポジトリルート基準の明示実行例、regression の注入、intake の案内のみという境界を確認。
- `git diff --check` — PASS。
- 変更パス確認 — すべて許可範囲 `tools/agent-project` 配下。`agent_project` パッケージ内と
  dashboard は未変更。

## 採用した前提・未解決事項・範囲外

- 完了条件は、`--config` 明示実行で生成・表示される両フックの推奨値が同じ routing 正本に基づき、
  README と CLI ヘルプから「何が自動で書かれ、何を手動設定するか」が一意に分かること、と解釈した。
- `codd_gate_regression.py` の既存責務と README の境界を優先し、`intake_cmd` の自動注入は行わない。
- 未解決事項なし。
- 範囲外で新たに見つけた問題なし。

# t10 成果報告

## サマリー

リポジトリは変更していない。対象の `docs/designs/codd-gate-design.md` は、書込許可された
`tools/agent-project/` の外にあるため、§4.1を直接更新できなかった。

現行実装との突合で、§4.1が追随すべき境界は次のとおり。

- `codd_gate_detect.py`、`codd_gate_status.py`、`codd_gate_routing.py`、
  `codd_gate_base.py`、`codd_gate_debt.py`、`codd_gate_wiring.py`、
  `codd_gate_regression.py` はパッケージ外の sibling 部品として残す。
- 責務は検出、利用可否判定、推奨文字列、YAMLの冪等注入、および任意のCLI所見に限る。
- `codd_gate_wiring.py` の実測入口は `probe_wiring()`。フック契約名
  `detect_wiring` はその別名であり、零設定の sibling 自動走査には意図的に選ばれない。
- package doctor へ所見を出す場合だけ、設定へ
  `hooks: {wiring: codd_gate_wiring}` を明示する。設定しない場合は
  `python3 tools/agent-project/codd_gate_wiring.py --config .agent/agent-project.yaml`
  を直接実行する。このCLIは読み取り専用で、YAMLを書き換えない。
- `codd_gate_wiring.py` は `upsert_yaml_text()` と `apply_yaml_file()` をライブラリ関数として持ち、
  明示された `regression_cmd` と `intake_cmd` を冪等注入できる。
  `codd_gate_regression.py` のCLIは `regression_cmd` だけを冪等注入する。
- `agent_project/*` への再結合と dashboard 変更は対象外。

slop-policeで§4.1を点検した結果、更新時は「自動検出」「配線層」などの抽象語だけで済ませず、
上記の選択条件と実行コマンドを明記するのが妥当。現状の
`codd_gate_wiring.py` 行にある `detect_wiring()` という関数表記と、
「値をディスクへ残す唯一の経路は `codd_gate_regression.py`」という記述は実装と一致しない。

## 検証

- `python3 -m unittest tools/agent-project/tests/test_codd_gate_wiring.py tools/agent-project/tests/test_codd_gate_regression.py`
  - 67 tests、すべて成功。
- `git status --short`
  - 作業開始時・検証前ともに差分なし。
- `agent_project/hooks.py`、`codd_gate_wiring.py`、`codd_gate_regression.py`、
  README、GUIDE、関連テストを読み合わせた。

## 前提・未解決事項

- 完了条件は、§4.1を現行の明示有効化境界へ合わせること、と解釈した。
- 「変更してよいのは `tools/agent-project` 配下のみ」を、対象文書の指定より優先した。
- 未解決: 許可範囲外の `docs/designs/codd-gate-design.md` §4.1は未更新。

@followup docs/designs/codd-gate-design.md の書込権限を持つタスクで、上記境界へ§4.1を更新する。

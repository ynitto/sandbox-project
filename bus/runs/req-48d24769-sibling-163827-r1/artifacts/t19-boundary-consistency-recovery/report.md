# t19 sibling 境界整合修復

## 成果

- `codd_gate_regression.py` から重複していた YAML 行編集・書込み API を削除した。生成 CLI は
  `codd_gate_wiring.upsert_yaml_text()` と `apply_yaml_file()` だけを使う。
- 生成 CLI を `probe_wiring()` へ接続した。codd-gate の未検出、バージョン不適合、repos schema
  不適合、`verify` capability 不足では YAML を変更せず終了コード 3 を返す。
- `README.md` に正準手順を追加した。稼働診断は `agent-project doctor`、パッケージ外の結線診断は
  読み取り専用の `codd_gate_wiring.py --config ...` を使う。
- `test_codd_gate_regression.py` は重複書込み API の不在と、version、schema、capability の
  fail-closed を固定した。
- 変更は `tools/agent-project/README.md`、`codd_gate_regression.py`、
  `tests/test_codd_gate_regression.py` の3ファイルだけ。`agent_project/` と dashboard は未変更。

## 検証

- 指定完了条件:
  `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py' && grep -nE 'codd_gate_regression|regression_cmd|intake_cmd' tools/agent-project/README.md && ! grep -nE 'build_config.*メモリ上で自動|_apply_codd_gate_auto_wiring' tools/agent-project/README.md`
  - 終了コード 0。codd-gate 関連 105 tests 成功。
- sibling 境界:
  `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  - 終了コード 0。
- 除外範囲:
  `git diff --quiet -- tools/agent-project/agent_project tools/agent-dashboard tools/agent-project/dashboard`
  - 終了コード 0。
- `git diff --check`
  - 終了コード 0。

## 前提と未解決事項

- t16 報告には永続化を `codd_gate_regression.py` に寄せる案もあったが、このタスクの具体指定
  「`codd_gate_wiring.py` の冪等注入経路へ一意化」を優先した。`codd_gate_regression.py` は
  利用者向け生成 CLI のまま、書込み実装だけを wiring に委譲する。
- `docs/designs/codd-gate-design.md` は許可された書込範囲外なので変更していない。§4.1 の強い保証
  （version、schema、capability 不適合時は書かない）へ実装を合わせた。
- 範囲外で別タスク化が必要な問題は見つからなかった。

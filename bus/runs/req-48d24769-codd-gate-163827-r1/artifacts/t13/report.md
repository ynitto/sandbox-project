切り口: 永続化の所有者を一文で固定し、人専有の設定ファイルを `agent_project` 本体が書かない境界まで言い切る。

## 成果候補

`docs/designs/codd-gate-design.md` §4.1 の「永続化（値をどこへ残すか）」冒頭は、次の一文を正典とする。

> codd-gate 固有設定の永続化は任意 sibling 部品の `codd_gate_regression.py` が担い、`agent_project` 本体は codd-gate 固有設定を生成・注入・永続化しない。

現 HEAD の同段落には、`codd_gate_regression.py` が yaml へ書き込む唯一の主体であること、Config 生成による自動配線がないこと、`agent_project` 本体が値を永続化しないことが、すでに上記と同じ強さで記載されている。重複追記はせず、作業ツリーを変更しなかった。

## 検証内容と結果

- §4 と §4.1 を確認し、永続化の主体が `codd_gate_regression.py` 一つに限定され、`agent_project` 本体の Config 生成は値を自動注入しないと明記済みであることを確認した。
- `tools/agent-project/codd_gate_regression.py` の `upsert_config_text()` と `apply_to_file()` を確認し、同モジュールが yaml への冪等な書き込みを実装していることを確認した。
- `tools/agent-project/tests/test_codd_gate_regression.py` を確認し、冪等更新、既存内容の保持、利用不能時の no-op が検証対象であることを確認した。
- `git diff` と `git status --short --untracked-files=all` は空で、指定 worktree に変更がないことを確認した。文書の現状確認だけなのでテストとリンタは実行していない。

## 前提・未解決事項・範囲外

- 完了条件は、§4.1 だけで「永続化の担当は `codd_gate_regression.py`」「`agent_project` 本体は永続化しない」の二点を判断できることとした。
- 対象文書は許可された `tools/agent-project/` 配下ではないため、書込制約を優先した。現 HEAD が完了条件を満たすため、適用作業も不要と判断した。
- `agent_project`、dashboard、テストは変更していない。範囲外で別タスク化が必要な問題は見つからなかった。

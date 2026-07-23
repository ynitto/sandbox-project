切り口: 三つの検出語を同じパッケージ境界と一つの否定コマンドに固定し、判定対象のずれをなくす。

## 成果候補

`docs/designs/codd-gate-design.md` §4.2「境界の完了条件（決定的ゲート）」の正典コマンドを、次へ置き換える。

```sh
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

設計上の完了条件は、このコマンドが exit 0 を返すこととする。検索先をパッケージ本体の
`tools/agent-project/agent_project` に限るため、許可された sibling 部品
`tools/agent-project/codd_gate_*.py` は違反扱いしない。`_apply_codd_gate`、`_codd_gate`、
`import codd_gate` のいずれかがパッケージ内に残れば `git grep` が成功し、先頭の `!` によって受入は失敗する。

作業ツリーは変更していない。指定された唯一の編集対象
`docs/designs/codd-gate-design.md` が、許可された書込範囲 `tools/agent-project/` の外にあるためである。

## 検証内容と結果

- 現行 §4.2 を確認した。正典のコマンドは
  `! git grep _apply_codd_gate -- tools/agent-project` であり、三つの検出語を
  `tools/agent-project/agent_project` に対して一括検証する条件にはなっていない。
- 上記の成果候補コマンドを現 HEAD で実行し、exit 1（受入失敗）を確認した。
  `configfile.py`、`doctor.py`、`model.py` に対象語が残っているためで、否定条件は残存を正しく検出している。
- `git status --short --untracked-files=all` は出力なし。リポジトリ内の変更はない。
- 文書候補のみのため、テスト、リンタ、型チェックは実行していない。

## 前提・未解決事項・範囲外

- 完了条件は、三つの検出語を別々の参考コマンドへ分散させず、同じ対象パスに対する一つの否定形
  `git grep` で検証できることと解釈した。
- 現行文にある「`_apply_codd_gate` は `_codd_gate` に包含されるため連結は冗長」という説明より、
  要求された三つの禁止対象をコマンド上で明示することを優先した。
- 設計書への反映には書込範囲の変更が必要であり、本タスクでは未反映である。
- agent-project、dashboard、テストの実装変更と、検出された残存コードの撤去は範囲外。

@followup 許可範囲に `docs/designs/codd-gate-design.md` を加え、§4.2 の正典コマンドと説明を上記候補へ置き換える。

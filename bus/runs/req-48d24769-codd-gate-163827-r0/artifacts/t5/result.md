# t5 成果報告

## 成果

作業ツリーは変更していない。正典 `docs/designs/codd-gate-design.md` §4.1 には、次の境界がすでに明記されており、依頼の完了条件を満たしている。

- `agent_project` パッケージは `codd_gate_*` を import せず、Config/Charter/Task 型とも結合・依存しない。
- `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品で、欠落・削除してもパッケージの挙動は変わらない。
- 自動検出レイヤの責務は sibling 側に閉じる。`codd_gate_detect.py` が実体を検出し、`codd_gate_status.py` が使用可否を判定し、`codd_gate_routing.py` が実引数を組み立てる。パッケージ起動時の自動配線はしない。

対象文書は書込許可された `tools/agent-project/` の外にあるため、既存記述への重複追加も行っていない。

## 検証

- `.codegraph/` がないことを確認し、通常検索で §4 と §4.1 を照合した。
- §4.1 の該当記述を行 278、280、323 で確認した。
- `git diff --check` は成功し、`git status --short` は空だった。
- 実装検索では `agent_project/configfile.py`、`doctor.py`、`model.py` に `codd_gate_*` の自動配線・import が残っていることを確認した。設計上の目標境界と現行実装は一致していない。

## 前提・未解決事項・範囲外

- 前提: 「§4.1 に明記する」は、同等の規範がすでに明記されていれば追加編集を要しない、と解釈した。
- 前提: 「変更してよいのは `tools/agent-project` 配下のみ」を対象文書の編集指示より強い制約として扱った。
- 未解決事項: 文書の追加表現を必須とする場合は、`docs/designs/codd-gate-design.md` の編集権限を持つタスクが必要。
- 範囲外: agent-project と dashboard の実装・テストは変更していない。

@followup 別タスクで `agent_project/configfile.py`、`doctor.py`、`model.py` の `codd_gate_*` 自動配線・import を除去し、§4.1 の境界へ実装を合わせる。

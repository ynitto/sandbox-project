# t17 境界整合修正

## 判定

未完了。指定 worktree の許可範囲内では、README と GUIDE の診断入口、および重複 API の再導入を防ぐテストを更新した。統合先にだけ存在する削除対象と、許可範囲外の設計書は変更していない。

## 成果

- `tools/agent-project/README.md` に
  `codd_gate_wiring.py --config .agent/agent-project.yaml` を読み取り専用の正準診断手順として追加した。
- `tools/agent-project/GUIDE.md` も direct CLI を唯一の正準診断入口と明記した。
- `tools/agent-project/tests/test_codd_gate_wiring.py` に、`upsert_yaml_text` と
  `apply_yaml_file` を wiring モジュールへ再導入させない回帰テストを追加した。
- `agent_project/` パッケージ内と dashboard は変更していない。

## 検証

- `PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'`
  - 成功: 113 tests
- t16 の指定 README grep と禁止 grep
  - 成功
- `git diff --check`
  - 成功

## 前提と未解決事項

- 指定 worktree の `HEAD` は `77389919`、統合先 `ap/sibling-163827` は `fdb66cb4` だった。
  checkout、merge、branch 操作は禁止されているため、指定 worktree の基点を変更しない前提を採った。
- 削除対象の `codd_gate_wiring.upsert_yaml_text` と `apply_yaml_file` は指定 worktree には存在せず、
  後続の統合コミット `eab4161e` にだけ存在する。この基点からは削除差分を作れない。統合先へ本成果を
  反映した後、同 API と `TestYamlInjection` を削除する必要がある。
- `docs/designs/codd-gate-design.md` は変更許可された `tools/agent-project/` の外にあるため変更していない。
  §4.1 のモジュール表から重複 API と yaml 注入責務を削除し、生成 CLI の保証を
  「codd-gate の実体未検出なら書き込まない」に直し、package doctor 経路の記述を除く必要がある。

@followup 統合先 `fdb66cb4` を基点にした専用 worktree と `docs/designs/codd-gate-design.md` の書込許可を付けて、残る削除と §4.1 更新を実施する。

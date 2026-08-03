切り口: 同一 status レコードを budget 無しから有りへ変換し、旧 reader の射影結果が完全不変であることを1件で固定する。

## 成果／サマリー

- `status-reader-optional-budget.patch` に、`tools/agent-project/tests/test_coordination.py` へ追加する契約テスト1件を作成した。
- テストは `_peer_nodes()` で budget 欠落時の結果を取得後、同じ `status/<node>.json` に optional `budget` を additive に追加し、読取結果が完全一致することを確認する。
- reader 本体は `updated_iso`、`fresh_after_sec`、`node` のみを投影し未知キーを既に無視するため、変更不要と判断した。変更候補はテスト1ファイルのみで、表示仕様、必須フィールド、他 reader・ビューには触れない。

## 検証内容と結果

- 参照 `main` (`09b7f9d28dad41343dbb9aa586d12b68a6d37536`) に対する `git apply --check`: PASS。
- パッチを一時コピーへ適用後、`PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests -p 'test_coordination.py'`: PASS（35 tests）。
- 同じ隔離手順で `PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests`: PASS（終了コード0）。
- 参照 worktree の `git status --short`: 変更なし。

## 前提・未解決事項・範囲外

- 前提: 本タスクの「status reader 結合点」は scope 内の `agent_project/coordination.py::_peer_nodes()` と解釈した。`agent-dashboard` は out of scope の別ツールとして変更しない。
- 前提: 本契約の責務は optional な外側 `budget` の存在を旧 reader が無視できること。budget block 全体の schema 準拠は schema／fixture 担当の契約で固定するため、ここでは最小の代表値 `{"can_accept": true}` を使う。
- 未解決事項なし。
- 範囲外で見つけた問題: 機械ゲートが参照する `test_codd_gate_wiring.py` は参照 main に存在しない。本変更では修正しない。

{"ok": true, "issues": []}

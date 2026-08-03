切り口: 新契約の全9フィールドを、未超過かつ受付可能な一貫した正常系として1件で網羅する fixture。

## 成果／サマリー

- `tools/agent-project/tests/fixtures/status-node-with-budget.json` として統合する生成物を `status-node-with-budget.json` に作成した。
- 既存 reader が使う `node`, `availability`, `updated_iso`, `fresh_after_sec` を維持し、optional な `budget` block を additive に追加した。
- `budget` は t3 の `node-budget-summary.schema.json` に定義された `can_accept`, `exceeded`, `soft`, `on_exhausted`, `spent_min`, `limit_min`, `spent_tokens`, `token_limit`, `period` をすべて含む。

## 検証内容と結果

- `python3 -m json.tool status-node-with-budget.json`: PASS。
- Python stdlib による t3 schema の型・非負制約・enum との照合: PASS。
- fixture の budget キー集合と schema の properties キー集合の完全一致: PASS。
- 参照 repo の `tests/_shared.py::mk_peer()` と `agent_project/loop.py::write_status()` を静的突合し、既存 status フィールド名・型に一致することを確認した。
- 変更は本 artifacts/t4 内の fixture 1件と本報告だけ。schema、reader、本番コード、他 fixture、テスト基盤、参照 repo は変更していない。
- 環境に `jsonschema` パッケージがないため外部 validator は未実行。新依存を追加せず、上記の stdlib 照合で代替した。

## 前提・未解決事項・範囲外

- 前提: t3 成果の schema を本タスクの正準契約とし、後続の統合先は t2 指定の `tools/agent-project/tests/fixtures/status-node-with-budget.json` とした。
- 前提: 固定 `updated_iso` は再現可能な fixture 用であり、鮮度を検証するテストでは一時配置時に現在時刻へ差し替える。
- 前提: `can_accept: true`, `exceeded: false`, `soft: false` は未超過の正常系として意味的にも整合する。
- 未解決: 元要求の schema 配置 `schemas/...` と分解タスクの `tools/agent-project/schemas/...` の不一致は統合側で確定が必要。本タスクでは schema を変更していない。
- 範囲外で発見: 現行 `write_status()` はまだ `budget` を生成しない。writer 実装・契約テスト側の担当で統合が必要。

{"ok": true, "issues": []}

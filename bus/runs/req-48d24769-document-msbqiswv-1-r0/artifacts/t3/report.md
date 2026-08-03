切り口: 既存 `_node_budget_state()` の8項目を改名せず保持し、受付判断 `can_accept` だけを加える最小の additive 契約。

## 成果／サマリー

- `tools/agent-project/schemas/node-budget-summary.schema.json` として統合する生成物を `node-budget-summary.schema.json` に作成した。
- Draft 2020-12、basename の `$id`、snake_case、非負の消費量・上限、既存 enum に合わせた。
- budget block 自体と各フィールドは optional、`additionalProperties: true` とし、旧 status reader と将来のフィールド追加を妨げない。
- 既存射影の `exceeded`, `soft`, `on_exhausted`, `spent_min`, `limit_min`, `spent_tokens`, `token_limit`, `period` に `can_accept` を追加した。全19行で約30行以内。

## 検証内容と結果

- `python3 -m json.tool`: PASS（JSON 構文）。
- Python stdlib によるキー・型・enum・非負制約・`additionalProperties: true` の明示検査: PASS。
- `wc -l`: 19行。上限内。
- 参照 repo の `schemas/node-budget.schema.json`、`schemas/agent-control.schema.json`、`tools/agent-project/agent_project/prioritize.py::_node_budget_state()` と静的突合した。
- out_of_scope に従い、status 生成処理、fixture、reader、テストは変更していない。

## 前提・未解決事項・範囲外

- 前提: タスク固有 scope の `tools/agent-project/schemas/...` を統合先とし、生成物は中間成果物プロトコルに従って本ディレクトリへ置く。参照 repo は読み取り専用のため変更しない。
- 前提: optional 互換性は既存 `agent-control` の budget block と同様、`required` を置かず未知キーを許容することで表す。
- 前提: `can_accept` の算出規則は schema の責務外。統合側では既存実行意味に合わせ、通常は `!exceeded || on_exhausted == "degrade"` とするのが妥当。
- 未解決: 元要求のルート `schemas/...` と本タスク scope の `tools/agent-project/schemas/...` に配置差がある。本成果では明示 scope を優先した。
- 範囲外で発見: 現行 writer は budget block をまだ生成しない。後続タスクで additive に埋め込む必要がある。

{"ok": true, "issues": []}

# synth result

- 統合差分: `node-budget-summary-integration.patch`
- 正準 schema: `schemas/node-budget-summary.schema.json`
- writer: `write_status()` が既存 `_node_budget_state()` の8項目を再利用し、`can_accept` を加えた optional `budget` を出力する。予算未設定時はキーを出さない。
- fixture/契約: writer 出力を schema と `status-node-with-budget.json` に照合する。旧 `readNodeStatuses()` は budget 無し／有りで射影が完全一致する。
- 非採用: t3 の `tools/agent-project/schemas/` 配置と、t5 の内部 `_peer_nodes()` のみを対象にしたテストは、元要求および gate と矛盾するため統合しなかった。
- 境界: `AGENT_CONTROL_DIR/status/<tool>-<pid>.json` の process heartbeat は別契約のため変更なし。

## verification

- JSON 構文: PASS
- `test_commands.py`: 97 tests PASS
- `liveness-host.test.js`: 8 tests PASS
- agent-project 全テスト: PASS
- `git diff --check`: PASS
- target checkout への `git apply --check`: PASS

{"constraints":["node-budget-summary の正準スキーマはリポジトリ直下 schemas/node-budget-summary.schema.json に置き、tools/agent-project/schemas 配下を正準にしない","project root の status/<node>.json に加える budget は optional かつ additive とし、出力時は node-budget-summary schema に準拠させる","旧ビュー readNodeStatuses() は budget の有無で既存の射影結果を変えず、両入力を同じ契約テストで固定する","node budget の値は既存 _node_budget_state() を再利用して生成し、project root status と AGENT_CONTROL_DIR の process heartbeat を別契約として扱う"]}

class=propagate

{"constraints":["node-budget-summary の正準スキーマはリポジトリ直下 schemas/node-budget-summary.schema.json に置き、tools/agent-project/schemas 配下を正準にしない","project root の status/<node>.json に加える budget は optional かつ additive とし、出力時は node-budget-summary schema に準拠させる","旧ビュー readNodeStatuses() は budget の有無で既存の射影結果を変えず、両入力を同じ契約テストで固定する","node budget の値は既存 _node_budget_state() を再利用して生成し、project root status と AGENT_CONTROL_DIR の process heartbeat を別契約として扱う"]}

根拠: gate の3件は配置、writer、旧ビューの継続的な境界を指している。t3 の個別 scope は元要求と t1 の既存配置調査に反するため採用しない。`write_status()` への具体的な接続方法、テスト配置、環境の `yaml` 不足はこの変更だけの判断とする。

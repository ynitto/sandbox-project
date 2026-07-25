# gate2 検証結果

verify=pass

- `main...HEAD` は `tools/agent-project` 配下の7ファイルのみ。`git diff --check` 成功。
- 旧識別子 `_apply_codd_gate_auto_wiring`、`_codd_gate_wiring_module`、`doctor_codd_gate_findings`、`_codd_gate_debt_module` は0件。
- 契約は `wiring.detect -> detect_wiring`、`wiring.findings -> doctor_findings` で実装・provider alias・doctor・mock が一致。
- 明示 provider は `_hook_import` のみ、未指定は `_hook_scan_siblings` のみを通り、明示失敗時に暗黙 fallback しない。
- config の `hooks` 透過、doctor の判定値受け渡し／finding 透過／provider 不在時 no-op、regression の `run_verify` 呼出しと blocked/done、intake の汎用 JSON 契約を確認。
- 独立実行: 関連統合テスト 210/210、codd-gate provider 群 112/112、境界抜粋 16/16 成功。
- 依存報告の集計も整合（t8 の242件は関連210件に `test_codd_gate_regression` 32件を加えた値）。

{"ok": true, "issues": []}

# verify=fail

## 判定

t3/t4/t5 のそのままの統合は不許可。schema と fixture の値契約は整合するが、正準配置、production writer、旧ビュー reader の契約が未接続で、元要求の完了条件を満たさない。

## blocking issues

1. **schema の統合先が完了条件と不一致**
   - 場所: `artifacts/t3/report.md`（想定先 `tools/agent-project/schemas/node-budget-summary.schema.json`）
   - 実リポジトリに `tools/agent-project/schemas/` はなく、既存 schema と元完了条件の正準先はルート `schemas/`。
   - 修正: 成果物を `schemas/node-budget-summary.schema.json` へ統合する前提に直す。

2. **正準 writer が budget block を出力しない**
   - 場所: `tools/agent-project/agent_project/loop.py::write_status()`、`artifacts/t4/status-node-with-budget.json`
   - fixture は9項目を持つが、唯一の project-root status writer の `rec` に `budget` がなく、3成果物を統合しても実際の `status/<node>.json` は fixture の形にならない。
   - 修正: `_node_budget_state()` の8項目を再利用し `can_accept` を加えた optional `budget` を `write_status()` の共通 `rec` に接続し、writer 出力を fixture/schema と照合する契約テストを追加する。

3. **後方互換テストが旧ビュー reader を検証していない**
   - 場所: `artifacts/t5/status-reader-optional-budget.patch`、`tools/agent-dashboard/src/features/agent-project/main/project.js::readNodeStatuses()`
   - patch は agent-project 内部の `_peer_nodes()` だけを検証する。元完了条件の「旧ビュー」の実 consumer は agent-dashboard の `readNodeStatuses()`。
   - 修正: `tools/agent-dashboard/test/liveness-host.test.js` に budget 無し／有りの同一 status を与え、旧ビュー射影の完全一致を固定する。内部 reader テストを残す場合も、完全な9項目の budget と `can_accept: false` を使い、budget を判断に使う不正変更を検出できるようにする。

## 独立検算

- Draft 2020-12 meta-schema check: PASS
- t4 `budget` の t3 schema validation: PASS
- schema properties と fixture budget keys: 9/9 完全一致
- 既存 `_node_budget_state()` 8項目 + `can_accept`: 9/9 完全一致、重複なし
- 型・enum・非負制約の敵対値4件: 4/4 reject
- t5 patch の最新 main への `git apply --check`: PASS
- patch 適用後 `test_coordination.py`: 35 tests PASS
- patch 適用後 agent-project 全テスト: exit 0
- patch の変更範囲: `test_coordination.py` 1ファイル、追加テスト1件のみ
- agent-dashboard test: 環境に `yaml` module がなく起動不能。ただし該当契約テスト自体が patch に存在しないことは静的に確認済み。

## agent-reviewer 集約

`functional` / `test` / `architecture` の全 perspective が `REQUEST_CHANGES`。上記3点を blocking として採用した。

{"ok": false, "issues": ["t3 の schema 統合先が正準のルート schemas/ と不一致", "production write_status が budget block を出力せず fixture と実出力が未接続", "t5 が旧ビュー readNodeStatuses の budget 無し・有り契約を検証していない"]}

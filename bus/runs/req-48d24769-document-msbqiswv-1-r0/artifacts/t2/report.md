# t2 調査成果: fixture・契約テスト・codd-gate／機械ゲート

参照: `ynitto/sandbox` main `09b7f9d28dad41343dbb9aa586d12b68a6d37536`

## (a) 成果／サマリー

### 完了条件として採用した前提

- 本タスクの完了は、変更を加えることではなく、`tools/agent-project` に追加する代表 fixture と optional 後方互換テストの最小配置、および最終検証コマンドを既存構成から特定すること。
- 本番コード、schema、fixture、テストは変更せず、指定どおりテストも実行しない。
- graph の後続 t3/t4 が明示する `tools/agent-project/schemas/node-budget-summary.schema.json` と `tools/agent-project` 内の fixture を、この run の配置契約として優先する。
- 「旧 reader」は scope 内の `agent_project/coordination.py::_peer_nodes()` と解釈する。他ツールの reader／view は調査しない。

### 既存 fixture パターンと追加先

- `tools/agent-project` には、commit 済み JSON fixture、`tests/fixtures/`、共通 fixture loader のいずれも存在しない。
- status fixture の既存正準は `tests/_shared.py::mk_peer()`。各テストの `TemporaryDirectory` 内へ `status/<node>.json` を `json.dumps()` で生成する。
- reader 契約の既存配置は `tests/test_coordination.py::TestAtomicClaim`。隣接する `test_peer_liveness_uses_freshness_not_availability` が `mk_peer()` と `_peer_nodes()` を直接組み合わせている。
- writer 契約の既存配置は `tests/test_commands.py::test_node_status_writes_per_node_file` と `TestStatusHeartbeat::test_write_status_content`。`write_status()` の出力を `json.loads()` し、必要キーを明示 assert する。
- 後続 t4 は「fixture 1件」を明示しているため、最小配置は `tools/agent-project/tests/fixtures/status-node-with-budget.json`。fixture が1件だけなので loader/helper は新設せず、テストから `Path(__file__).parent / "fixtures" / "status-node-with-budget.json"` を直接読む。
- 代表 fixture は `node`、`availability`、`updated_iso`、`fresh_after_sec` と optional `budget` を持たせる。固定時刻は読取テスト内で現在時刻へ差し替え、鮮度依存の flake を避ける。`budget` 内部は t3 の schema を正として具体化する。

### 後方互換契約テスト

- `_peer_nodes()` は `node`、`updated_iso`、`fresh_after_sec` だけを読み、未知の `budget` を参照しない。`allocate_distributed_tasks()` も既存キーだけを読むため、additive な `budget` は本番 reader 修正なしで受容できる。
- 最小の新規テストは `tests/test_coordination.py` に1件。旧形式は既存 `mk_peer()` で作り、新形式は上記 fixture を一時 `status/` へコピーして、両 node が `_peer_nodes(cfg)` に現れることを同時に assert する。
- これにより `budget` の「欠落」と「存在」を同じ reader 契約で固定できる。既存戻り値に budget を露出させる変更や新しい reader abstraction は不要。
- schema と fixture の値契約もこの1件で、schema JSON の読込、`budget` のキー集合／型、fixture の代表値を明示 assert すればよい。既存規約は runtime `jsonschema` 依存を持たず、stdlib parser と明示的テストで schema を突き合わせる。

### codd-gate／機械ゲート定義

- agent-project は codd-gate を直接配線しない。`GUIDE.md` と `README.md` は、state repo の `./tools/check` にテスト・codd-gate 等を集約し、`regression_cmd: ./tools/check` と一度だけ設定する opt-in 契約を定義する。
- 汎用境界は `tests/test_config.py::TestGenericHookConfig`。`repos.json` の有無で自動配線せず、明示した `regression_cmd` / `intake_cmd` を変更せず通す。
- この作業 workspace の `project.json` にある機械ゲートは次の2本。
  1. `python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_wiring.py'`
  2. `test -n "${KIRO_BASE_REV:-}" && python3 tools/codd-gate/codd-gate.py verify --base "$KIRO_BASE_REV" && AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests`
- 参照 main には `tools/agent-project/tests/test_codd_gate_wiring.py` が存在しない。1本目は0件収集でも成功し得るため、この変更の完了根拠にはできない。2本目を最終機械ゲートとして使う。

### 後続が使う最小コマンド

リポジトリ root から実行する。

```bash
# schema と fixture が有効な JSON であること
python3 -m json.tool tools/agent-project/schemas/node-budget-summary.schema.json >/dev/null
python3 -m json.tool tools/agent-project/tests/fixtures/status-node-with-budget.json >/dev/null

# optional budget 有無、fixture/schema の明示契約、既存 reader の互換性
PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 \
  python3 -m unittest discover -s tools/agent-project/tests -p 'test_coordination.py'

# writer に budget を埋め込む統合差分がある場合のみ追加
PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 \
  python3 -m unittest discover -s tools/agent-project/tests -p 'test_commands.py'

# 最終機械ゲート（統合側が KIRO_BASE_REV を設定）
test -n "${KIRO_BASE_REV:-}" && \
  python3 tools/codd-gate/codd-gate.py verify --base "$KIRO_BASE_REV" && \
  AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests
```

## (b) 検証内容と結果

- 参照 repo root に `.codegraph/` が無いことを確認し、`tools/agent-project` の全ファイル名と関連参照を静的に照合した。
- `mk_peer()`、`_peer_nodes()`、status writer テスト、汎用 regression hook、workspace の `project.json` にある機械ゲートをソースで突き合わせた。
- 専用参照 worktree の `git status --short` は空で、参照 repo は変更していない。
- out-of-scope 指定に従い、テスト、JSON 検査、codd-gate は実行していない。上記コマンドの PASS 確認は final verify ノードが行う。

## (c) 採用した前提・未解決事項・範囲外事項

- 採用した前提: 静的 fixture の既存例が無いため、現行の一時 JSON fixture 流儀を維持しつつ、t4 の明示要件に限って `tests/fixtures/` を初回作成する。
- 未解決: 元要求の schema パス `schemas/node-budget-summary.schema.json` と graph の t3 パス `tools/agent-project/schemas/node-budget-summary.schema.json` が不一致。本 run では具体的な後続 scope である後者を採用したが、統合時に受入基準との整合確認が必要。
- 範囲外で発見: `test_codd_gate_wiring.py` を指す既存機械ゲートと参照 main のファイル構成が不整合。今回の schema／fixture変更とは別のゲート保守問題として扱う。
- 範囲外: agent-dashboard 等、他ツールの旧 view 互換は未調査・未保証。

{"constraints":["status fixture の既存正準は tests/_shared.py::mk_peer による TemporaryDirectory 内生成であり、静的 fixture loader は存在しない","optional budget は status reader の既存結果を変えず、欠落時も従来どおり読めることを1契約テストで固定する","agent-project は codd-gate を直接配線せず regression_cmd の汎用境界を維持する"]}

# t1 調査結果: status / budget / reader 結合点

## (a) 成果・サマリー

### 前提と完了判定

- このノードの完了条件は、実装変更ではなく、(1) `status/<node>.json` の唯一の生成経路、(2) 現行 budget 契約、(3) additive な新フィールドを無視または受容する reader の結合点を、後続 t3/t4/t5 がそのまま使える粒度で特定することと解釈した。
- 調査対象は `https://github.com/ynitto/sandbox` の `main` を専用 worktree で取得した内容。参照 worktree は変更していない。
- リポジトリに `tools/agent-project/schemas/` は存在しない。共通 JSON Schema の既存配置はリポジトリ直下 `schemas/` であり、元要求の完了条件も `schemas/node-budget-summary.schema.json` を指定している。後続は直下 `schemas/` を正とするのが妥当。

### 1. `status/<node>.json` の生成経路

正準 writer は `tools/agent-project/agent_project/loop.py` の次の一本だけ。

- `status_dir(cfg)`（18–20行）: `<project-root>/status`。
- `node_status_path(cfg)`（23–32行）: `Config.node` が空なら `None`、あれば板と共通の `normalize_node_id` を通し `status/<normalized-node>.json` を返す。
- `write_status(cfg)`（55–87行）: まず同一の `rec` を組み立て、後方互換の `<root>/status.json` と、node 指定時の `<root>/status/<node>.json` の双方へ同一 JSON 本文を書く。
- 現行 `rec` のキー: `host`, `watch`, `level`, `paused`, `node`, `availability`, `updated_iso`, `fresh_after_sec` と `detect_runtime()` が返す runtime 系キー。`budget` はまだ無い。
- 呼び出しは通常パス終端・pause・level 変更等（`loop.py` 553, 674, 714行）、project 遷移（`project.py` 637行）、command 処理（`commands.py` 939, 945行）、idle heartbeat（`loop.py` 90–102行）。従って `write_status` の `rec` に optional `budget` を一度加えるのが全生成経路を覆う最小の結合点。
- 注意: `rec` に加えると `<root>/status.json` にも同じ `budget` が additive に載る。両ファイルを同一スナップショットに保つ現行設計に沿うため、`status/<node>.json` だけ別本文に分岐する根拠はない。

既存 writer 契約テストは `tools/agent-project/tests/test_commands.py::test_node_status_writes_per_node_file`（286–300行）。現在は node 名とファイル名正規化だけを固定している。

### 2. 現行 budget 契約

契約は三層あり、混同しないこと。

1. 正準データ契約: `schemas/node-budget.schema.json`
   - `$AGENT_BUDGET_DIR`（既定 `~/.agents/budget/`）の `config.json` と `ledger/<YYYYMMDD>.jsonl` を定義。
   - config は `version`, 全体/ワークロード別の分上限、token 上限、allocation/computed/rates 等。全 object は原則 `additionalProperties: true` で additive 互換。
   - ledger は `ts`, `workload`, `seconds` 必須。token 実測値が無ければ reader が rates で推定する。

2. agent-project の集計射影: `tools/agent-project/agent_project/prioritize.py::_node_budget_state()`（365–433行）
   - 設定無し、または全上限 0 なら `None`（無制限）。
   - 非 `None` 時の現行戻り値は `exceeded`, `soft`, `on_exhausted`, `spent_min`, `limit_min`, `spent_tokens`, `token_limit`, `period`。
   - `project` ワークロードを対象に、時間上限と token 上限を AND 的に併用し、全体またはワークロード実効上限のどれか到達で `exceeded=true`。
   - この戻り値が新しい node-budget-summary の最も近い既存射影であり、重複集計を作らず再利用すべき結合点。

3. 別用途の agent-control heartbeat: `schemas/agent-control.schema.json#/$defs/status_record` と `prioritize.py::_write_status()`（527–549行）
   - 置き場所は `$AGENT_CONTROL_DIR/status/agent-project-<pid>.json` で、今回の project root `status/<node>.json` とは別物。
   - 既存 `budget` block は `{exceeded: boolean, soft: boolean}` のみを写す。schema は `budget` 自体 optional、かつ `additionalProperties: true`。
   - この二値 block は管理面のプロセス別 heartbeat 用なので、そのまま node summary の正準 schema と見なさない。ただし `exceeded` / `soft` の名称と additive 方針は互換上維持する価値がある。

後続タスク `document-msbqiswy-3` は `budget_summary.can_accept` と呼んでいる一方、元要求は `status/<node>.json` の `budget` キーを指定している。今回の契約では外側キーを `budget` とし、その中のフィールドとして `can_accept` を置く解釈が元要求と整合する。`budget_summary` という二つ目の外側キーを増やす判断は本タスクでは行わない。

### 3. reader の結合点と additive 互換性

agent-project 内部 reader:

- `tools/agent-project/agent_project/coordination.py::_peer_nodes()`（29–55行）は JSON object から `updated_iso`, `fresh_after_sec`, `node` だけを読む。未知の `budget` を無視するため、追加後も現行挙動は不変。
- 同 `allocate_distributed_tasks()`（298–345行）は `updated_iso`, `fresh_after_sec`, `node`, `availability` だけを読む。未知の `budget` を無視する。将来 `can_accept` を割当適格性へ反映する正しい結合点は 307–318行の eligible 構築部。
- `claim_distributed_task()`（174行以降）は現在 node status を読まない。将来 claim 時にも予算を再確認するなら、status 読取/鮮度判定を共通 helper に抽出して allocation と共有する必要があるが、今回の optional 互換契約には変更不要。

旧ビュー reader（実体は agent-dashboard）:

- `tools/agent-dashboard/src/features/agent-project/main/project.js::readNodeStatuses()`（1291–1311行）が `status/<node>.json` の旧ビュー結合点。
- 入力 record の必要キーだけから新しい表示用 object を作るため、未知の `budget` を自動的に捨てる。budget が欠落していても存在していても、現行出力 shape は同一で壊れない。
- 契約テストの最小配置は既存の `tools/agent-dashboard/test/liveness-host.test.js` の `readNodeStatuses` テスト（124–143行）。同じ入力を旧形式（budget 無し）と新形式（budget 有り）で読み、`deepStrictEqual` で投影結果が同一と固定できる。reader 本体修正は不要。
- `<root>/status.json` 用 `readStatus()`（1278–1286行）は `{...rec, ageSec, fresh}` と spread するため、同じ writer から `budget` が載った場合もそのまま受容・保持する。

### 後続への最小実装指針

- t3: schema は既存規約どおりリポジトリ直下 `schemas/node-budget-summary.schema.json`。射影 object 単体を定義し `additionalProperties: true`、新フィールドは optional にする。
- t4: `write_status()` が生成する代表 record へ optional `budget` を追加した fixture とする。集計は `_node_budget_state()` を再利用する。
- t5: 旧ビュー互換は `readNodeStatuses()` の入力に budget 有/無を与え、出力が同じであることを固定するだけで足りる。本体修正は不要。

## (b) 検証内容と結果

- CodeGraph: 参照 repo root に `.codegraph/` が無いため未使用。`rg` と行番号付きソース確認で生成/読取の全参照を追跡した。
- `PYTHONDONTWRITEBYTECODE=1 python3 -m unittest tools/agent-project/tests/test_commands.py tools/agent-project/tests/test_coordination.py`: **130 tests, OK**。writer と内部 reader 周辺の既存契約を確認。
- `node --test tools/agent-dashboard/test/liveness-host.test.js`: **未実行相当（環境依存で起動失敗）**。参照 worktree に依存 `yaml` がインストールされておらず `MODULE_NOT_FOUND`。代替として `readNodeStatuses` の実装と既存テストを静的突合した。
- 参照 worktree の `git status --short`: 空。テスト後も変更無し。
- 本タスクの scope に従い、schema・fixture・本番 reader・テストは変更していない。

## (c) 採用した前提・未解決事項・範囲外事項

- 採用した前提: 「旧ビュー」は project root の node status を読む agent-dashboard `readNodeStatuses` を指す。agent-project 内部 reader も併記した。
- 未解決: node-budget-summary の最終フィールド集合は本タスクの範囲外。既存射影の8フィールドと、後続要求で明示された `can_accept` が候補だが、schema 作成ノードが確定する。
- 範囲外で発見: 後続要求の呼称 `budget_summary.can_accept` と元要求の外側キー `budget` に表記揺れがある。外側キーを増やさず `budget.can_accept` とするのが最小かつ元要求準拠。
- 範囲外で発見: project root の `status/<node>.json` 自体には現時点で専用 JSON Schema が無い。新 schema は budget block 単体の射影契約として追加するのが既存構成への最小変更。

{"constraints":["status/<node>.json の新 budget 情報は optional・additive とし、旧 reader が必要キーだけを投影できる契約を維持する","node budget の集計は既存 _node_budget_state() を再利用し、別の台帳 reader を新設しない","project root status/<node>.json と AGENT_CONTROL_DIR/status/<tool>-<pid>.json は別契約として扱う"]}

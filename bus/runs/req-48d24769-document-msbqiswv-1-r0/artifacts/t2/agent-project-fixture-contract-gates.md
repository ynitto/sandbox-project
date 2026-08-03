# t2 調査成果: agent-project の fixture・契約テスト・機械ゲート

## 完了条件として採用した前提

- このノードの完了は、`tools/agent-project` 内だけを読み、(1) status JSON fixture の既存流儀、(2) optional な追加フィールドの後方互換性テスト配置、(3) codd-gate を含む最小の完了判定コマンドを特定して、後続ノードがそのまま実装・検証に使える形で渡すこと。
- 本番コード、テスト、fixture、schema は変更しない。指定どおりテストも実行しない。
- 「旧 reader」は、この scope 内では `agent_project/coordination.py` の status reader を指す。agent-dashboard の旧ビューは別ツールなので調査対象外。
- 参照したのは `https://github.com/ynitto/sandbox` の `main` を規定スクリプトで作成した読み取り専用 worktree。共有 checkout への書き込み、commit、push はしていない。

## 特定した既存パターン

### 1. status fixture

- `tools/agent-project` には `tests/fixtures/` や commit 済み JSON fixture は存在しない。
- 既存の正準パターンは `tests/_shared.py:95-112` の `mk_peer()`。各テストの `TemporaryDirectory` 内に `status/<node>.json` を `json.dumps()` で生成する。
- writer の既存契約テストは `tests/test_commands.py:286-300` の `TestCommandsIngest.test_node_status_writes_per_node_file`。`write_status()` が `status.json` と `status/pc-a.json` を書くこと、node 名を保つことを固定している。
- 最小の追加方法は新しい fixture 階層を作らず、`mk_peer()` に optional な `budget` 引数を追加し、指定時だけレコードへ `budget` を足すこと。未指定時の既存 JSON は変えないため、旧 fixture も同じ helper で表現できる。

### 2. optional フィールドの後方互換性契約

- scope 内の reader は `agent_project/coordination.py::_peer_nodes()`（29-55行）と `allocate_distributed_tasks()` 内の reader（303-320行）。どちらも `updated_iso`、`fresh_after_sec`、`node`、後者のみ `availability` を読む。未知のトップレベル `budget` は参照せず無視する。
- 置き場所は `tests/test_coordination.py` の `TestAtomicClaim`。隣接する `test_peer_liveness_uses_freshness_not_availability`（76-87行）が `mk_peer()` で status reader の契約を固定している。
- 最小の新規テストは 1 件でよい。budget なしの peer と、新 budget 付き peer を同じ一時 `status/` に置き、`_peer_nodes(cfg)` が両方を同じく返すことを assert する。これで「新フィールドが存在しても欠落していても壊れない」を一つの reader 契約として固定できる。
- writer 側の schema 準拠確認は既存 `test_node_status_writes_per_node_file` に budget の値を 1 assert 追加するのが最短。ただし schema validator が既存依存に無いため、テスト基盤や新依存を増やさず、JSON Schema 自体の構文確認は標準ライブラリ、値の形は明示的 assert で固定する。

### 3. codd-gate・機械ゲート

- agent-project は codd-gate を直接配線しない。`README.md:354-363` と `GUIDE.md:127-129` が、state repo の `./tools/check` にテスト・codd-gate 等を集約し、`regression_cmd: ./tools/check` を一度だけ設定する契約を定義している。
- 汎用性の契約テストは `tests/test_config.py::TestGenericHookConfig`（621行以降）。`repos.json` があっても自動配線しないこと、明示した `regression_cmd` / `intake_cmd` を変更せず通すことを固定している。
- 回帰ゲートの実行契約は `tests/test_autonomy.py::TestLoopEngineering` の 3 テスト（500-547行）。失敗時 block、成功時 done、実行 cwd が workspace clone ではなく project workdir であることを固定している。
- doctor の provider 境界は `tests/test_doctor.py:300-323`。provider 出力の保持と provider 不在時の no-op を固定している。
- 現在の project 機械ゲート `project.json` には次の 2 定義がある。
  - codd-gate 配線の個別ゲート: `python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_wiring.py'`
  - 設計・コード整合の全体ゲート: `test -n "${KIRO_BASE_REV:-}" && python3 tools/codd-gate/codd-gate.py verify --base "$KIRO_BASE_REV" && AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests`
- ただし参照した `main` には `tools/agent-project/tests/test_codd_gate_wiring.py` が存在しない。個別ゲートは現状ではテスト 0 件になり得るため、この変更の完了根拠には使えない。

## 後続が使う最小コマンド

作業ツリー root から次の順で実行する。

```bash
# schema JSON の構文（新依存なし）
python3 -m json.tool tools/agent-project/schemas/node-budget-summary.schema.json >/dev/null

# writer fixture 契約
PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 \
  python3 -m unittest discover -s tools/agent-project/tests -p 'test_commands.py'

# optional field を無視する reader 契約
PYTHONDONTWRITEBYTECODE=1 AGENT_FLOW_STUB_SLEEP_MAX=0 \
  python3 -m unittest discover -s tools/agent-project/tests -p 'test_coordination.py'

# project の最終機械ゲート（KIRO_BASE_REV は統合側が設定）
test -n "${KIRO_BASE_REV:-}" && \
  python3 tools/codd-gate/codd-gate.py verify --base "$KIRO_BASE_REV" && \
  AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests
```

`test_codd_gate_wiring.py` が追加されない限り、存在しないファイルを pattern にした個別ゲートは完了判定から外す。codd-gate 自体は最終機械ゲートで実行する。

## 検証内容と結果

- `tools/agent-project` 配下の全ファイル名を列挙し、静的 fixture / `tests/fixtures` が無いことを確認した。
- `fixture`、`status/<node>.json`、`budget`、`contract`、`codd-gate`、`regression_cmd`、テストコマンドを横断検索し、上記の writer・reader・helper・gate 契約をソースで突合した。
- project の機械ゲート定義と、参照 `main` 上の対象テストファイル有無を突合し、`test_codd_gate_wiring.py` 不在を確認した。
- 指示に従いテスト・codd-gate は実行していない。したがって実行結果の PASS は未確認であり、上記コマンドを統合・検証ノードが実行する必要がある。

## 未解決事項・範囲外で見つけた問題

- 元要求は `schemas/node-budget-summary.schema.json`、分解済み t3 は `tools/agent-project/schemas/node-budget-summary.schema.json` を指定しており、schema 配置が不一致。t2 では変更せず、最小コマンドは t3 の scope に合わせた。統合時に正準パスを一つへ確定する必要がある。
- 旧 agent-dashboard view の互換性は別ツール配下であり未調査。scope 内で保証できるのは agent-project の status reader のみ。
- `test_codd_gate_wiring.py` を指す機械ゲートが実ファイルと不整合。今回の schema/fixture 契約とは別のゲート保守問題として扱う。

{"constraints":["tools/agent-project の status テスト fixture は静的 fixtures ディレクトリではなく tests/_shared.py::mk_peer と TemporaryDirectory で生成する","agent-project は codd-gate を直接配線せず regression_cmd: ./tools/check の汎用境界を維持する","未知の optional budget フィールドは status reader が無視し、budget 欠落時の既存結果を維持する"]}

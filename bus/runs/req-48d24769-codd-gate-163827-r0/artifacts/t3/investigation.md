# t3 調査結果: codd-gate 境界の現行実装

## 前提・完了条件

- 調査対象は指定 worktree の現行コード、とくに `tools/agent-project/`。正典との比較には `docs/designs/codd-gate-design.md` §4/§4.1 と `docs/designs/agent-project-design.md` §4.1 を参照した。
- 完了条件は、E1–E3 の汎用フック、`codd_gate_regression.py`、自動検出の対象と経路、yaml/CLI による有効化経路、`_apply_codd_gate`・`_codd_gate`・`import codd_gate` 系の残存箇所を列挙し、正典との差を明示すること。変更は行わない。
- 「残存箇所」はテスト・任意 sibling も検索対象に含めるが、境界違反の判定では `agent_project/*`（パッケージ）と sibling/test を区別した。

## サマリー

E1–E3 の汎用フックはすでに存在し、コマンド文字列だけで外部 CLI を差し込める。一方、現行の `agent_project/*` には codd-gate 固有の起動時自動配線、doctor 検出、debt パーサ import、コマンド名のハードコードが残る。これは正典の「パッケージは汎用フックのみ」「パッケージから `codd_gate_*` を import・自動配線しない」「Config 生成で値を埋めない」と不一致である。

## E1–E3 の汎用フック

| Hook | 現行の入口と実行点 | 契約・挙動 |
|---|---|---|
| E1 task verify | `agent_project/verify.py:6` `run_verify()`、`agent_project/mr.py:457` `run_verify_stable(task.verify, ...)` | shell command、cwd は `verify_cwd` / workspace clone / `workdir`、`KIRO_BASE_REV` を環境へ渡し、exit 0 のみ PASS。未定義 verify は FAIL。 |
| E1 charter acceptance | `agent_project/charter.py:221` で `## acceptance` を読む。`agent_project/project.py:26` `evaluate_acceptance()` が各行を `run_verify_stable()` で実行し、同 `:239` の evaluate が結果を収束判定へ渡す。 | 全 acceptance PASS が project done の根拠。自然言語行は `resolve_charter_acceptance()` で実行可能 verify へ合成し、未解決なら done にしない。 |
| E2 regression | `Config.regression_cmd` (`agent_project/config.py:122`)。`agent_project/mr.py:461-473` で task verify PASS 後・done 確定前に `run_verify(cfg.regression_cmd, cfg.workdir, ..., verify_env)`。 | タスク非依存の横断ゲート。非 0 なら block、人判断へ。`regression_revert` は任意。 |
| E3 intake | `Config.intake_cmd` (`agent_project/config.py:124`) と `run_intake()` (`agent_project/model.py:518`)。パス開始時は `agent_project/mr.py:558`、watch idle は `agent_project/loop.py:610`。 | cwd=`workdir`、timeout 付き shell command。stdout の object/array JSON を `id` で冪等取り込み。失敗・非 JSON は journal に記録してループを維持。`intake_interval` で律速。 |

## yaml / CLI の有効化経路

- 共通の優先順位は `CLI > config file > CONFIG_DEFAULTS`。`resolve_config()` は `agent_project/configfile.py:185`、既定は `regression_cmd=None`, `intake_cmd=None` (`:117-120`)。
- E2: yaml/json の `regression_cmd:`、または共通 CLI `--regression-cmd` (`agent_project/configfile.py:493`)。`--regression-revert` も同所。
- E3: yaml/json の `intake_cmd:` / `intake_interval:`、または `--intake-cmd` / `--intake-interval` (`agent_project/configfile.py:497-502`)。
- E1 task verify: backlog markdown の `- verify:`、または `enqueue --json` の task spec。E1 acceptance: charter markdown の `## acceptance`。
- `agent-project.yaml.example:171-180` は汎用 regression と codd-gate intake のコメント例を持つ。README は codd-gate 用 E2/E3 値を例示する。
- 正典上の明示的な永続化 CLI は `python3 tools/agent-project/codd_gate_regression.py --config ...`。現行 `install.sh` は sibling の zipapp 同梱と codd-gate の隣接インストールを行うが、この生成 CLIを実行して yaml を更新する経路はない。

## `codd_gate_regression.py`

- sibling CLI であり、パッケージ型には依存しない。`build_regression_cmd()` は usable status のとき `codd-gate verify --base "$KIRO_BASE_REV" --repos ...` を生成する。
- `upsert_config_text()` は生テキストを最小差分で `regression_cmd` 1行へ冪等 upsert し、既存コメントを保持する。`cmd=None` は完全 no-op。`apply_to_file()` と CLI main が永続化する。
- CLI は `--config`, `--codd-gate`, `--repos`, `--base`, `--dry-run` を持つ。`--codd-gate` の解決順は explicit → PATH → sibling の `tools/codd-gate/codd-gate.py`。
- 対象は E2 の `regression_cmd` のみで、`intake_cmd` と acceptance は生成・注入しない。
- 正典との差: §4.1 は生成可否に実在・version・schema・capability の判定を記すが、現行 main は `codd_gate_status.detect_status()` を使い、実在だけを確認する。version/schema/capability を実測する `codd_gate_wiring.detect_wiring()` は使っていない。また config に `root:` が無い場合の既定 repos は `.agent-project/repos.json` であり、CLI 既定 config `.agent/agent-project.yaml` とは別概念なので、実際の root が異なる場合は `--repos` が必要。

## 自動検出の対象と現行経路

任意 sibling の検出部品は次の分担になっている。

- `codd_gate_detect.py`: 実体を explicit → PATH → `tools/codd-gate/codd-gate.py` で解決。`--version`、repos JSON 最小構造、`--help` による `verify` / `tasks` / `--debt` capability を個別プローブ。
- `codd_gate_status.py`: `CoddGateStatus` と no-op 縮退。未検出、version 不明/下限未満、schema 不適合を findings にする。ただし `detect_status()` 単体は実在しか測らない。
- `codd_gate_wiring.py`: 上記プローブを `detect_wiring()` で結合し、既存 E2/E3 文字列の結線判定と推奨文字列を返す。
- package の現行起動時経路: `build_config()` (`agent_project/configfile.py:376`) → `_apply_codd_gate_auto_wiring()` (`:201`) → `repo_registry_path(cfg)` が検出した `<root>/repos.yaml|yml|json` → `_codd_gate_wiring_module()` → `detect_wiring()`。E2/E3 の片方または両方が空なら、推奨値をメモリ上の `cfg` へ補う。両方が明示済み、repos 不在、sibling 不在、未検出/非互換/capability 不足では no-op。
- doctor の現行経路: `collect_doctor_signals()` (`agent_project/doctor.py:528`) → `doctor_codd_gate_findings()` → `detect_wiring()`。これは読み取り専用だが package が codd-gate を名指しする。
- install の対象: `install.sh:46-52` が全 `codd_gate_*.py` を zipapp ルートへコピーし、`:64-72` が隣接 `tools/codd-gate/install.sh` を任意実行する。

## 残存箇所

### package 内（正典の境界と衝突）

- `_apply_codd_gate_auto_wiring`: `agent_project/configfile.py:201` 定義、`:376` 呼び出し。`build_config()` が E2/E3 を動的補完する直接の不一致。
- `_codd_gate_wiring_module`: `agent_project/doctor.py:287` 定義。`configfile.py:220` と `doctor.py:314` が使用。
- `doctor_codd_gate_findings`: `agent_project/doctor.py:309` 定義、`:528` で常時 doctor 集約へ追加。
- `import codd_gate_wiring`: `agent_project/doctor.py:295,303`。
- `_codd_gate_debt_module`: `agent_project/model.py:494` 定義、`:552` 使用。
- `import codd_gate_debt`: `agent_project/model.py:504,512`。汎用 E3 が codd-gate 出力だけを特別扱いする。
- その他の名指し: `agent_project/verify.py:9` のコメント、`:356` の `_KNOWN_COMMAND_WORDS` に `codd-gate`、`model.py` / `mr.py` / `charter.py` / `configfile.py` の例・説明。import/自動配線ほど強い結合ではないが、「パッケージから名指ししない」を字義どおり受け入れるなら残存対象。

### sibling / tests（任意部品として妥当、または撤去対象実装に追随するテスト）

- sibling import は `codd_gate_regression.py:40-42`, `codd_gate_status.py:27`, `codd_gate_wiring.py:39-47`。これらは `tools/agent-project/` 直下の任意部品同士の依存。
- unit test import は `tests/test_codd_gate_{debt,detect,regression,routing,wiring}.py`。
- package 自動配線を固定するテストは `tests/test_agent_project.py:3877-3944` (`TestCoddGateAutoWiring`)。境界修正時はこの期待値が正典と逆になる。
- plain module 名そのものの `import codd_gate`（suffix なし）は 0 件。残っているのはすべて `codd_gate_*` import。

## 検証内容と結果

- `.codegraph/` が無いため、CodeGraph は使用せず `rg` と `git grep` で全 `tools/agent-project` を検索した。
- `git grep '_apply_codd_gate'`: package 2件（定義・呼び出し）。
- `git grep '_codd_gate'`: package の configfile / doctor / model と、関連 test/docstring を確認。
- `git grep -E '^[[:space:]]*(from|import)[[:space:]]+codd_gate'`: package production import は `doctor.py` 2箇所と `model.py` 2箇所。ほかは sibling/test。
- 関連単体テスト4ファイルを実行: 72 tests、全 PASS。
- `git status --short`: 出力なし。指定 worktree のファイル変更は 0 件。

## 未解決事項・範囲外

- 設計正典と実装・README の主張はすでに食い違っている。どちらを採るかは元要求どおり正典 §4/§4.1 の境界を採用した。
- dashboard / agent-project 実装 / テストの変更は行っていない。
- @followup: 実装タスクでは package の自動配線・doctor 固有検出・debt 固有パースを sibling/明示 CLI 側へ戻し、`TestCoddGateAutoWiring` と README の「起動時自動検出」記述を正典へ合わせる必要がある。

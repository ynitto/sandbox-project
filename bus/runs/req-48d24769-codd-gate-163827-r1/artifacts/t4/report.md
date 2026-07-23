# t4 現行コード調査報告

## サマリー

調査対象は worktree の HEAD `abe5906a18149a1ab4a4c97654bfa7d7950eb763` にある
`tools/agent-project/agent_project` と、その呼び出し先である sibling
`tools/agent-project/codd_gate_*.py`。実装変更はしていない。

現行コードには E1〜E6 の汎用フックがある。一方、`agent_project/*` には次の codd-gate
専用経路も残っている。

1. `build_config()` が `_apply_codd_gate_auto_wiring()` を必ず呼び、条件が揃うと
   `cfg.regression_cmd` と `cfg.intake_cmd` をメモリ上で補う。
2. `run_intake()` が sibling `codd_gate_debt` を import し、存在すれば専用パーサを使う。
3. `cmd_doctor()` が sibling `codd_gate_wiring` を import し、codd-gate 専用 finding を足す。

したがって、正典 `docs/designs/codd-gate-design.md` §4 の
「パッケージは汎用フックのみ」「sibling を削除しても同一挙動」という目標境界にはまだ達していない。
§4.2 の受入 `! git grep _apply_codd_gate -- tools/agent-project` も現状は失敗する。

## 関数単位の差し込み点

行番号は上記 HEAD のもの。

| 境界 | 設定・入力 | 実行境界 | 呼び出し元と結果 |
|---|---|---|---|
| E1 タスク verify | `model.py:68 parse_task()` が `- verify:` を `Task.verify` に読む | `mr.py:409 _settle_task()` → `verify.py:41 run_verify_stable()` → `verify.py:6 run_verify()` | `run_verify()` が `shell=True`、対象 task の `vcwd`、`verify_timeout`、`KIRO_BASE_REV` 付きで実行。exit 0 が PASS |
| E1 charter acceptance | `charter.py:201 parse_charter()` が `## acceptance` を読む | `project.py:69 resolve_charter_acceptance()` → `project.py:233 _project_evaluate()` → `project.py:26 evaluate_acceptance()` → `run_verify_stable()` | 明示 `verify_cwd`、単一 repo の一時 clone、`workdir` の順で cwd を決める。全件 PASS が project done の根拠 |
| E2 regression | `configfile.py:493 _add_common()` の `--regression-cmd`、`configfile.py:117 CONFIG_DEFAULTS`、`config.py:122 Config.regression_cmd` | `_settle_task()` の task verify PASS 後 → `run_verify(cfg.regression_cmd, cfg.workdir, ..., verify_env)` | task の一時 clone ではなく常に `cfg.workdir` で実行。act 前 HEAD を `KIRO_BASE_REV` に渡す。失敗時は `_block()`、任意で `_revert_workdir()` |
| E3 intake | `configfile.py:497 _add_common()` の `--intake-cmd`、`configfile.py:501` の interval、`config.py:124-125` | `mr.py:553 _run_setup()` → `model.py:518 run_intake()`。さらに `loop.py:580 run_watch()` の idle poll からも直接呼ぶ | `cfg.workdir` で `shell=True` 実行。timeout は `verify_timeout`。exit 非 0、例外、非 JSON は journal に残して空結果。成功 spec は `enqueue_task()` へ渡す |
| E4 push intake | `model.py:428 ingest_inbox()`、`model.py:236 enqueue_task()`、CLI `enqueue` | `_run_setup()` が `run_intake()` の直後に `ingest_inbox()` | codd-gate を知らない汎用の JSON/Markdown 入力境界 |
| E5 notify | `needs.py:565 notify()` | `loop.py:283 run_loop()` の終了処理から呼ぶ | `notify_cmd` の stdin に digest。失敗してもループを落とさない |
| E6 executor | `configfile.py:438 _add_common()` の `--executor` | `flow.py:81 build_agent_flow_cmd()` が agent-flow の `--executor` に渡し、`flow.py:620 act_via_agent_flow()` が実行 | 実行バックエンドの汎用差し替え口 |

### E2 の実際の呼び出し境界

```text
run_loop()
  -> _settle_task()
     -> run_verify_stable(task.verify, task側cwd, ..., task側KIRO_BASE_REV)
     -> task verify が安定 PASS かつ regression_cmd が truthy
        -> run_verify(regression_cmd, cfg.workdir, ..., act前HEADのKIRO_BASE_REV)
           -> FAIL: _block()、done/review へ進めない
           -> PASS: protect / progress / flake 判定へ進む
```

`regression_cmd` 自体の実行部は汎用で、codd-gate の import はない。
`mr.py` と `verify.py` に codd-gate の名称を含むコメントや許可コマンド一覧はあるが、
実行時の型・関数依存ではない。

### E3 の実際の呼び出し境界

```text
run_loop()
  -> _run_setup()
     -> run_intake()

run_watch() の idle loop
  -> run_intake()

run_intake()
  -> interval 判定（_INTAKE_LAST、backlog パス単位）
  -> subprocess.run(intake_cmd, cwd=workdir, timeout=verify_timeout)
  -> _codd_gate_debt_module()
     -> あれば parse_debt_output()
     -> なければ json.loads() の汎用パース
  -> enqueue_task()
```

設計書の「S0・watch idle」という記述どおり、各 `run_loop()` の開始時と watch idle の
両方で呼ばれる。同一プロセス内では `_INTAKE_LAST` が interval を共通に律速する。

ただしパース境界は汎用ではない。`model.py:494 _codd_gate_debt_module()` が
`sys.path` に `tools/agent-project` を足して sibling を import する。sibling がある場合は
不正レコードを件単位で隔離し、ない場合は非 dict を読み飛ばすだけなので、sibling 削除前後で
厳密には挙動が同じではない。

## 自動検出と自動配線

### 起動時の実行経路

```text
cli.py:4 main()
  -> configfile.py:190 resolve_config()
     CLI > 設定ファイル > CONFIG_DEFAULTS
  -> configfile.py:232 build_config()
     -> Config(regression_cmd=..., intake_cmd=...)
     -> configfile.py:201 _apply_codd_gate_auto_wiring()
        -> charter.py:326 repo_registry_path()
        -> doctor.py:287 _codd_gate_wiring_module()
        -> codd_gate_wiring.py:138 detect_wiring()
           -> codd_gate_detect.resolve_codd_gate()
           -> get_version()
           -> check_repos_schema_compat()
           -> detect_capabilities()
           -> codd_gate_wiring.judge_wiring()
        -> 推奨値で cfg の空フィールドを個別に上書き
```

`agent_project/__init__.py` は断片を一つの共有 global 名前空間へ `exec` する。
`doctor.py` が `configfile.py` より先に合成されるため、`configfile.py` は自ファイルに定義のない
`_codd_gate_wiring_module()` を呼べる。通常の Python モジュール import 境界ではなく、
共有名前空間に依存した呼び出しである。

自動配線の条件は次のとおり。

- 両コマンドが truthy なら検出自体を省略する。
- 片方だけ設定済みなら、その値は保持して空いている側だけを補う。
- `repo_registry_path()` が既存の `repos.json/yaml/yml` を見つけなければ検出しない。
- sibling 不在、バイナリ未検出、version/schema/capability 不適合なら空のまま。
- 書き換えるのは現在の `Config` オブジェクトだけ。yaml には永続化しない。

`charter.py:370 export_repo_registry()` は `load_charter()` 内の
`_apply_repo_registry()` から呼ばれる。一方、`build_config()` は `load_charter()` より前に終わる。
そのため、charter から初めて `repos.json` を生成する起動では自動配線が間に合わず、次回起動から
検出対象になる。既存レジストリがある場合だけ同じ起動で配線される。

### doctor の別経路

```text
cmd_doctor()
  -> doctor_codd_gate_findings()
     -> _codd_gate_wiring_module()
     -> detect_wiring(...)
     -> doctor_findings(...)
```

これは設定値を書かない読み取り経路だが、パッケージから sibling を直接 import する専用結合である。
`cmd_doctor()` は codd-gate finding を通常の doctor finding に無条件で連結する。

### sibling 側の境界

| 関数 | 役割 |
|---|---|
| `codd_gate_detect.py:39 resolve_codd_gate()` | explicit → PATH → 同梱 `tools/codd-gate/codd-gate.py` の順で実体を解決 |
| `codd_gate_wiring.py:138 detect_wiring()` | 実在、version、repos schema、capability の実測を直列化 |
| `codd_gate_wiring.py:104 judge_wiring()` | 既存コマンドの結線判定と推奨値生成 |
| `codd_gate_wiring.py:65/74 recommend_*()` | E2/E3 に渡す文字列を生成 |
| `codd_gate_debt.py:78 parse_debt_output()` | E3 stdout をレコード単位で正規化 |
| `codd_gate_regression.py:59/99/151` | `build_regression_cmd()` → `upsert_config_text()` → `apply_to_file()` で yaml に永続化 |

`codd_gate_regression.py` の永続化 CLI はパッケージから呼ばれていない。目標境界に残せるのは、
この明示実行による yaml 更新と、yaml/CLI から汎用 E1〜E3 へ入る経路である。

## 目標境界との差分

正典 §4 を満たすために実装側で切る必要がある境界は、調査上は次の3か所に限定できる。
このタスクでは変更していない。

| 現行の専用結合 | 目標境界との衝突 | 汎用フックへの影響 |
|---|---|---|
| `build_config()` → `_apply_codd_gate_auto_wiring()` | 起動時 Config 生成が codd-gate を名指しして値を注入する | E2/E3 の設定読込と実行部は独立している |
| `run_intake()` → `_codd_gate_debt_module()` | E3 の受け側が供給元固有パーサを import する | JSON object/array → `enqueue_task()` の汎用経路は残せる |
| `cmd_doctor()` → `doctor_codd_gate_findings()` | doctor が sibling の実在と専用状態を知る | 一般 doctor/audit finding には影響しない |

関連テストも現行専用結合を仕様として固定している。
`tests/test_agent_project.py:3877 TestCoddGateAutoWiring` は `build_config()` の自動注入を期待し、
`TestIntake.test_run_intake_one_bad_record_does_not_block_the_rest` は専用パーサ経路を期待する。
境界変更時に実装だけ消すと、この2群が失敗する。

## 検証

- `.codegraph/` は worktree に存在しなかったため、CodeGraph は使わず `rg` とソース読解で
  定義・全呼び出し元を追った。
- `python3 -m unittest -v tests.test_codd_gate_wiring tests.test_codd_gate_debt
  tests.test_agent_project.TestIntake tests.test_agent_project.TestCoddGateAutoWiring`:
  41件すべて PASS。
- regression hook の失敗、成功、workspace clone と `workdir` の cwd 分離を検証する3件:
  すべて PASS。
- `git grep -E 'codd_gate|codd-gate' -- tools/agent-project/agent_project`:
  専用 import/呼び出しを `configfile.py`、`doctor.py`、`model.py` で確認。
- §4.2 の受入相当 `! git grep _apply_codd_gate -- tools/agent-project`:
  FAIL。定義と `build_config()` からの呼び出しが残る。
- 調査前後の `git status --short`: 出力なし。対象 worktree は clean。

## 前提・未解決事項・範囲外

- 「現行コード」は依存タスク t2 が示した未 rebase の HEAD `abe5906a…` とした。最新 main は
  HEAD の祖先ではないため、この報告は main の後続変更を含まない。
- 「hooks」は正典の E1〜E6 を指すと解釈し、E1〜E3を詳述しつつ、E4〜E6も実在確認した。
- コメントに codd-gate の例があるだけの箇所と、実行時に sibling を import・呼び出す箇所を分けた。
  境界違反の判定対象は後者。
- 設計書や agent_project、dashboard、テストは変更していない。
- 範囲外で見つけた問題: 正典 §4/§4.1 は自動配線なしと断定しているが、現行コードとテストは逆の
  挙動を固定している。また、初回の charter 由来 `repos.json` 生成と自動配線には一起動分のずれがある。
  実装タスクではこの2点を同時に解消しないと、完了判定と表示が再び食い違う。

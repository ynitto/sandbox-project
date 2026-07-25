# t1〜t3 突合検証

verify=pass

## 独立再導出

- 対象 worktree の HEAD は `d6183170`、`git status --short` は空。
- `.codegraph/` は無いため、`rg`、Python AST、現物読解、git 履歴の読み取りで検証した。
- 受入 grep
  `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  は成功した。
- 現行 `agent_project/**/*.py` では、codd_gate import、`_apply_codd_gate*`、
  `_codd_gate*` の定義・呼び出し・その他参照はいずれも 0 件。説明・例示としての
  `codd_gate` / `codd-gate` 文字列は t1 記載どおり 8 件だった。
- 履歴も抜き取り、旧 `_apply_codd_gate_auto_wiring` の呼び出し元が
  `configfile.build_config`、旧 `doctor_codd_gate_findings` の呼び出し元が
  `doctor.cmd_doctor`、旧 `_codd_gate_wiring_module` の呼び出し元が旧 auto-wiring と
  doctor finding 経路だったことを確認した。現行ではいずれも残存しない。

## 現行の全経路

| 境界 | 定義 | 呼び出し元 |
|---|---|---|
| provider 解決 | `hooks._hook_provider` | `hooks._hook_resolution_error`; `doctor.doctor_wiring_findings`（detect/findings の2能力） |
| hooks 設定 | `configfile._normalize_hooks` | `configfile.build_config` |
| doctor wiring | `doctor.doctor_wiring_findings` | `doctor.cmd_doctor` |
| regression command | `Config.regression_cmd` | `mr._settle_task` 内の `run_verify(..., cfg.workdir, ...)` |
| intake command | `model.run_intake` | `mr` の pass 開始経路、`loop` の watch idle 経路 |

既存フックは、診断用の `hooks` capability provider と、実処理用の
`regression_cmd` / `intake_cmd` の二層である。t1〜t3 の記述と一致し、追加の実行時
codd-gate 固有経路は見つからなかった。

## テストと境界

- t3 と同じ現行シャード:
  `tests.test_backlog tests.test_config tests.test_doctor tests.test_autonomy
  tests.test_codd_gate_wiring tests.test_codd_gate_regression`
  → **241 tests / OK**。
- 現行の非自動配線テスト:
  `tests.test_config.TestCoddGateNoAutoWiring`
  → **4 tests / OK**。
- sibling provider 境界の全 `test_codd_gate_*.py`
  → **111 tests / OK**。
- `git diff --exit-code main...HEAD -- tools/codd-gate` は成功し、
  `git status --short -- tools/codd-gate` も空。`tools/codd-gate` への変更は無い。
- `main...HEAD` の差分自体が空で、スコープ外・無関係差分の混入は無い。

## issues

- (minor) t2 の `tests/test_agent_project.py -k CoddGateNoAutoWiring` による
  「4 tests / OK」は現行 HEAD では再現不能。同ファイルは削除・分割済みなので、
  証跡を `python3 -m unittest tests.test_config.TestCoddGateNoAutoWiring`
  （現行で 4 tests / OK）へ更新する。
- (minor) t3 の本文で provider 境界テストとして列挙されているのは wiring/regression のみ。
  完全な境界一覧には `test_codd_gate_base.py`、`test_codd_gate_debt.py`、
  `test_codd_gate_detect.py`、`test_codd_gate_routing.py` も追記する。今回は
  `test_codd_gate_*.py` 全111件成功で補完確認済み。
- (minor) t3 が記載済みのとおり、`doctor_wiring_findings` の generic provider 統合テストは
  `tests/test_doctor.py` に無い。後続では `km._hook_provider` を能力別 fake provider に差し替え、
  引数転送、片側欠落、provider 例外、明示解決失敗 warning を固定する。

{"ok": true, "issues": ["(minor) t2 の tests/test_agent_project.py 実行証跡は現行 HEAD で再現不能。tests.test_config.TestCoddGateNoAutoWiring の4件成功へ更新する", "(minor) t3 の provider 境界テスト一覧に test_codd_gate_base/debt/detect/routing.py を追記する（全 test_codd_gate_*.py 111件は成功済み）", "(minor) tests/test_doctor.py に doctor_wiring_findings の generic provider 統合テストを追加する"]}

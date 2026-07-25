# agent-project テスト影響調査

## サマリー

`tests/test_agent_project.py` は現行 HEAD (`d6183170`) で削除・分割済みである。直前
HEAD (`999d9fb7`) の旧ファイルと現行シャードを照合した結果、今回の変更で直接見るべき
移設先は次の5ファイルである。

| 関心事 | 現行の主テスト | 実装上の差し込み点 | 維持すべき観測可能な振る舞い |
|---|---|---|---|
| intake / model | `tests/test_backlog.py:180-254` | `agent_project/model.py::_parse_intake_records`, `run_intake`; `loop.py` の pass 開始・idle 呼出し | object/array JSON を受理する。現役 backlog の同一 id は重複投入しない。interval 内は再実行しない。空出力、非 JSON、非0終了、実行例外はループを落とさず0件。無効レコードだけ journal に記録し、正常レコードは継続投入する。run 開始時に取り込み、watch idle から次 pass を起こす。 |
| 自動配線しないこと | `tests/test_config.py:561-600` | `agent_project/configfile.py::build_config` | `repos.json` の有無だけで `regression_cmd` / `intake_cmd` をメモリ上で補完しない。明示コマンドは無変更で通す。 |
| 汎用フック解決 | `tests/test_codd_gate_wiring.py:280-356` | `agent_project/hooks.py::_hook_provider`, `_hook_resolution_error`, `HOOK_CAPABILITIES` | 本体は能力キー `wiring.detect` / `wiring.findings` と契約メソッド `detect_wiring` / `doctor_findings` のみを知る。明示設定は prefix より full key が優先。明示 module の解決失敗時は別 provider へ黙って fallback しない。未指定・provider 不在は no-op。cache は `None` も保持する。 |
| doctor | provider 側の所見は `tests/test_codd_gate_wiring.py:252-277`。本体 doctor の一般テストは `tests/test_doctor.py` | `agent_project/doctor.py::_hook_misconfig_findings`, `doctor_wiring_findings`, `cmd_doctor` | provider 未指定/不在は無所見。明示設定ミスだけ config/warn。detect/findings の片方欠落、import 例外、provider 呼出し例外は doctor 全体を落とさず no-op。両能力が揃えば cfg の `regression_cmd`, `intake_cmd`, `repo_registry_path`, 注入された `which`/`run` を detect に渡し、provider の findings を統合する。 |
| regression gate | `tests/test_autonomy.py:499-535` | `agent_project/mr.py` の done 確定直前の `run_verify(cfg.regression_cmd, cfg.workdir, ...)` | task verify が通った後だけ実行。成功なら done、失敗なら blocked。workspace task の一時 clone ではなく常に `cfg.workdir` で実行する。既存の `regression_revert` 挙動も変更しない。 |

## 変更対象と mock 対象

### 変更優先度: 高

1. `tests/test_config.py:561-600`
   - `TestCoddGateNoAutoWiring` と
     `test_configfile_has_no_codd_gate_auto_wiring_hook` は旧 private 名
     `_apply_codd_gate_auto_wiring` の「不存在」を直接検査している。
   - この1件は改名後の private 名を追うテストにせず、既存3ケース
     （repos 有/無で未補完、明示値を温存）を汎用 provider 名で表現するのがよい。
     private symbol の grep をテスト契約にしない。
   - `hooks:` の設定値が `Config.hooks` へ渡ること、および不正型が安全に空へ正規化されることは
     現在テストが無く、`configfile.py::_normalize_hooks` の最小追加テスト候補。

2. `tests/test_doctor.py`
   - 現在、`doctor_wiring_findings` 自体の本体側テストが無い。
   - 旧固有 mock 名 `_codd_gate_wiring_module` /
     `doctor_codd_gate_findings` を使うテストを移植・追加する場合の mock 対象は
     `km._hook_provider`。`side_effect` で detect module と findings module を能力キー別に返す。
   - fake provider の観測点は以下で十分:
     - `detect_wiring(**kwargs)` が受けた `regression_cmd`, `intake_cmd`,
       `repos_path`, `which`, `run`
     - `doctor_findings(judgment)` が同じ opaque judgment を受けること
     - provider 不在/片側欠落/例外で `[]`
     - 明示解決失敗では warn 1件
   - `cmd_doctor` の集約を直接調べる必要がある場合は `km.doctor_wiring_findings` が新しい
     patch 対象。旧 `km.doctor_codd_gate_findings` は使わない。

3. `tests/test_backlog.py:180-254`
   - テストの期待値は既に汎用で、実装変更時も維持する。
   - コメント/クラス説明の codd-gate 固有例は一般の「外部 detector/provider」に置換可能。
   - subprocess mock は現在使わず、短い `printf`/`true`/`false` を黒箱実行している。このままの方が
     shell/cwd/exit code を含む observable contract を保てる。
   - `_parse_intake_records` の直接テストが無い。改修時は最小ケースとして
     object/array、非 object 混在、title 欠落、数値 id、空白 id を追加候補とする。

### 維持（provider 固有テストとして隔離）

- `tests/test_codd_gate_wiring.py`
  - codd_gate provider 自身の検出・推奨コマンド・所見生成は provider 固有テストなので残す。
  - subprocess は `which=` / `run=` 注入で置換されている。
  - 唯一の `mock.patch` は `detect.Path.exists` (`:170`) で、同梱 binary 不在を再現するため維持。
  - `TestHookResolution` は本体の汎用契約も検査するが、現在は provider 固有モジュールを使う。
    本体の完全な非依存性を強めるなら、同じ契約の一時 fake module を使うテストを
    `test_hooks.py` 相当へ分離する候補。ただし今回の調査タスクでは変更しない。

- `tests/test_codd_gate_regression.py`
  - config ファイルへ codd-gate の推奨 `regression_cmd` を恒久注入する provider CLI のテスト。
    本体の実行時自動配線とは別責務なので維持。
  - mock 対象 `regression.detect_status` (`:352`, `:416`) は provider 内部に閉じており、
    agent-project 本体の改名対象ではない。

- `tests/test_autonomy.py:499-535`
  - gate の pass/fail は実 shell command (`true`/`false`) で観測するため mock 変更不要。
  - cwd 回帰の mock 対象 `km._task_verify_cwd` (`:532`) は workspace clone のみを偽装し、
    regression gate が `cfg.workdir` を選ぶことを黒箱確認しているため維持。
  - コメント中の codd-gate は例示に過ぎず、generic global hook の表現へ置換可能。

## 前提・未解決・範囲外

- 前提: 指定された `tests/test_agent_project.py` は分割前の論理名と解釈した。現行 HEAD では
  `d6183170` により削除済みで、直前コミットの旧ファイルを `git show` で読み、
  現行シャードへ対応付けた。
- 前提: この work は調査・対応表作成であり、実装・テストコード編集は後続タスクの担当。
  指定リポジトリには変更を加えていない。
- 未解決: 現行 `test_doctor.py` には generic wiring provider の本体統合テストが無い。
  provider 側 `render_findings` のテストだけでは、`doctor_wiring_findings` の引数転送、
  no-op 縮退、例外隔離を回帰検知できない。
- 未解決: `_normalize_hooks` と `_parse_intake_records` は統合経由の一部確認のみで、
  境界値の直接テストが不足している。
- 範囲外: `tools/codd-gate` の仕様、dashboard UI、設計文書は調査・変更していない。

@followup doctor の generic provider 統合テストを `tests/test_doctor.py` に追加し、旧固有 mock 名ではなく `km._hook_provider` / `km.doctor_wiring_findings` を patch 対象にする。

## 検証

- 実行:
  `python3 -m unittest tests.test_backlog tests.test_config tests.test_doctor tests.test_autonomy tests.test_codd_gate_wiring tests.test_codd_gate_regression`
- 結果: **241 tests / OK**（50.953秒）
- `git status --short`: 空。指定 worktree に変更なし。

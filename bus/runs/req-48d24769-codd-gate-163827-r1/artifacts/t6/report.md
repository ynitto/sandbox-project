# codd_gate sibling の責務分類

## 完了条件と前提

この調査の完了条件を、`tools/agent-project/codd_gate_*.py` の全7ファイルについて次の3点をコードで確認すること、と置いた。

1. 自動検出を実行するファイルと補助ファイルを分ける。
2. 推奨コマンドを生成するファイルと、設定へ書くファイルを分ける。
3. ディスクへの設定永続化を `codd_gate_regression.py` だけに置く根拠を、正典 §4、§4.1と実装の両方から確認する。

「永続化」は `.agent/agent-project.yaml` などの設定ファイルを作成または更新する処理を指す。`Config` オブジェクトへの一時的な代入、JSONの読み込み、subprocessによる能力照会は含めない。現行コードは worktree の HEAD `abe5906a18149a1ab4a4c97654bfa7d7950eb763` を対象とした。

## 分類結果

| ファイル | 分類 | 根拠 |
|---|---|---|
| `codd_gate_detect.py` | 自動検出の基本処理 | `resolve_codd_gate()` が explicit、PATH、同梱パスの順に実体を探す。`get_version()`、`check_repos_schema_compat()`、`detect_capabilities()` がバージョン、repos構造、サブコマンド対応を実測する。設定は書かない。 |
| `codd_gate_status.py` | 自動検出の判定補助 | `build_status()` が検出値を `CoddGateStatus` にまとめ、利用不能時は no-op に倒す。`detect_status()` は実体の存在だけを調べる簡易入口。設定は書かない。 |
| `codd_gate_wiring.py` | 自動検出の統合、結線判定、推奨コマンド生成 | `detect_wiring()` が detect/status を一続きに呼ぶ。`recommend_regression_cmd()` と `recommend_intake_cmd()` が E2/E3 用文字列を返し、`judge_wiring()` が推奨可否を決める。`doctor_findings()` は表示材料を返すだけで、yamlや `Config` を更新しない。 |
| `codd_gate_routing.py` | 推奨コマンド生成の補助 | `resolve_repos_arg()` などが `--repos` と `--repo-dir` の引数を組み立てる純粋関数。設定は書かない。 |
| `codd_gate_base.py` | regression実行値の補助 | `resolve_base_rev()` が `KIRO_BASE_REV`、task base、`HEAD~1` の順で基準revを選ぶ純粋関数。設定は書かない。 |
| `codd_gate_debt.py` | intake出力の正規化 | `parse_debt_output()` が stdout JSON を task spec に正規化する。検出、推奨生成、永続化のいずれも担当しない。 |
| `codd_gate_regression.py` | regression推奨値の生成と設定永続化 | `build_regression_cmd()` が E2 の値を作り、`upsert_config_text()` が既存テキストを最小差分で更新し、`apply_to_file()` と `main()` が `mkdir()`、`write_text()` を実行する。7ファイル中、書込み処理を持つ唯一のファイル。 |

責務の流れは次のとおり。

```text
codd_gate_detect.py
  -> codd_gate_status.py
  -> codd_gate_wiring.py
       -> 推奨 regression_cmd / intake_cmd（戻り値だけ）

codd_gate_status.py + codd_gate_routing.py
  -> codd_gate_regression.py
       -> regression_cmd を yaml へ明示的、冪等に保存
```

なお、現行 `codd_gate_regression.py` は `detect_wiring()` を呼ばず、`detect_status()` による実体確認だけで生成可否を決める。したがって上図の2系統は実装上は直結していない。

## 永続化を一ファイルに限る根拠

正典 `docs/designs/codd-gate-design.md` §4 は、`agent_project/*` が持つのは E1〜E6 の汎用フックだけで、codd-gate の値は人または明示的な install 手順が yaml/CLI に入れる、と決めている。§4.1 は機械的な書き手を `codd_gate_regression.py` 一つに限定し、起動時の `build_config()` による注入を禁止している。

この限定にはコード上の理由が3つある。

- `.agent/agent-project.yaml` は人が管理する設定である。明示起動された一つのCLIだけが書けば、いつ何を有効化したかをファイル差分で監査できる。
- sibling は任意部品であり、丸ごと削除してもパッケージの挙動を変えてはいけない。検出や推奨生成に書込みを混ぜると、doctorや起動時プローブが設定変更を伴い、この性質を失う。
- 推奨生成は環境依存の一過性の判定である。`codd_gate_wiring.py` が文字列を返すだけなら、未検出や非互換を no-op にできる。永続化は人が明示実行した `codd_gate_regression.py` の `upsert_config_text()` と `apply_to_file()` に集約でき、同じ値の再適用ではmtimeも変えない。

実装もこの境界をほぼそのまま表している。`codd_gate_*.py` に対する `write_text`、`mkdir`、`open`、`replace`、`unlink`、`rename` の検索では、実書込みは `codd_gate_regression.py:157-158` と `:184-185` だけだった。`codd_gate_detect.py` の `read_text()` は repos互換検査の読み取りである。`codd_gate_wiring.py` は regression/intake の両方を推奨するが、どちらも戻り値に留める。

依存タスク t4 が確認した `agent_project/configfile.py` の `_apply_codd_gate_auto_wiring()` は、yamlへの永続化ではなくプロセス内 `Config` の上書きである。ただし、正典が禁止する起動時自動配線には該当する。永続化の一意性が保たれていても、パッケージ境界はまだ満たしていない。

## 検証

- `rg --files tools/agent-project | rg 'codd_gate_.*\.py$'` で対象7ファイルを列挙した。
- 各ファイルの全関数定義と、書込みAPIの使用箇所を検索した。設定への実書込みは `codd_gate_regression.py` だけだった。
- 次の関連テストを実行し、81件すべてPASSした。

```text
python3 -m unittest -v \
  tests.test_codd_gate_detect \
  tests.test_codd_gate_wiring \
  tests.test_codd_gate_regression \
  tests.test_codd_gate_routing \
  tests.test_codd_gate_debt
```

- `TestRecommendedCommands` が E2/E3 の推奨文字列を、`TestApplyToFile` がファイル作成、更新、再実行時の無書込み、利用不能時の無作成を確認している。
- 最初のテスト指定では存在しない `tests.test_codd_gate_status` と `tests.test_codd_gate_base` も指定したため2件の import error になった。テストファイル一覧を確認して上記5モジュールに修正し、81件PASSを得た。status/base の専用テストファイルはないが、status は detect/wiring/regression/routing の各テストから間接検証されている。
- 調査前後の `git status --short` は空。対象 worktree は変更していない。

## 未解決事項と範囲外

- 正典 §4.1 は `codd_gate_regression.py` が detect/status/routing で実在、使用可否、実引数を解決すると説明する。一方、現行実装は `detect_status()` の存在確認しか使わず、`detect_wiring()` が確認するバージョン、schema、capabilityを永続化前に見ていない。文書と実装の差分だが、本タスクは調査のみのため直していない。
- `codd_gate_regression.py` が自動保存するのは `regression_cmd` だけである。`intake_cmd` の推奨値は `codd_gate_wiring.py` が生成するが、保存は人またはinstall手順に委ねられている。
- agent_project、dashboard、設計書、テストは変更していない。

@followup `codd_gate_regression.py` の生成可否を `detect_wiring()` 相当のバージョン、schema、capability判定へ揃えるか、正典の説明を存在確認だけに直すかを別タスクで決める。

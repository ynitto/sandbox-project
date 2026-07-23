# t5 成果報告

## サマリー

`docs/designs/codd-gate-design.md` §4.1 は、目標境界をほぼ書き切っている。設計上の結論は次のとおり。

- 自動検出は値を判断・生成するための読み取り専用レイヤであり、パッケージへの実行時配線ではない。
- `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品で、`agent_project/*` から import しない。
- 有効化は人または install 手順による明示設定だけ。E2/E3 は YAML または CLI、E1 acceptance は charter に書く。
- 機械による永続化は、人が明示実行する `codd_gate_regression.py` が YAML の `regression_cmd` 1 行を冪等更新する経路だけ。

ただし、§4.1 には説明不足があり、現物コードにも規範との不一致が残る。特に「生成時に何を検査するか」と「どの値をどこへ保存できるか」は、このままだと実装担当が違う解釈をする。

## 既存の責務記述

| 観点 | §4.1 が定める責務 | 根拠 |
|---|---|---|
| 自動検出レイヤ | `codd_gate_detect.py` が実体、version、repos schema、capability の生値を取得する。`codd_gate_status.py` が失敗を `usable=False` へ縮退する。`codd_gate_wiring.py` が実測を束ね、既存設定の結線判定と推奨文字列を返す。検出結果はプロセス内だけに置き、設定を変更しない。 | 設計書 284–299 行、`codd_gate_wiring.py` 138–172 行 |
| sibling 部品 | `codd_gate_detect/status/routing/base/debt/wiring/regression.py` はパッケージ外に置く。標準ライブラリだけを使い、Config/Charter/Task に依存せず、単体テストを持つ。削除してもパッケージの挙動を変えない。 | 設計書 252–255、273–290、322–327 行 |
| YAML/CLI 有効化 | `regression_cmd` と `intake_cmd` は人か install 手順が設定または CLI に明示する。未設定なら連携なし。起動時プローブで Config を補完しない。 | 設計書 256–259、301–304 行。CLI は `configfile.py` 493–500 行 |
| 永続化 | `codd_gate_regression.py` を人か install 手順が明示実行し、YAML の `regression_cmd` だけを最小差分で冪等更新する。使用不可なら既存ファイルを変えない。通常起動時には書かない。 | 設計書 306–320 行、`codd_gate_regression.py` 91–159、162–190 行 |

責務を実装判断用に並べ直すと、次の境界になる。

| 値 | 一時的な有効化 | 永続的な有効化 | 生成ツール |
|---|---|---|---|
| E2 `regression_cmd` | `--regression-cmd` | `agent-project.yaml` | `codd_gate_regression.py` が 1 行を生成・更新 |
| E3 `intake_cmd` | `--intake-cmd` | `agent-project.yaml` | なし。人または install 手順が値を書く |
| E1 acceptance | なし | charter の `## acceptance` | なし。人または install 手順が値を書く |
| E1 task verify | enqueue/task spec の `verify` | backlog/task spec | codd-gate の `tasks` 出力が供給しうるが、§4.1 の YAML 生成対象ではない |

## 不足と不一致

### 設計書で補うべき点

1. **有効化先の区別**

   301–304 行は `regression_cmd`、`intake_cmd`、acceptance をまとめて「yaml か CLI に書く」としている。しかし acceptance の正位置は charter の `## acceptance` であり、acceptance 用 CLI 設定はない。上表のように値ごとの保存先と一時指定方法を分けて書く必要がある。

2. **永続化対象の限定**

   256–259 行の「値を書く主体は一つだけ」は、`regression_cmd` と `intake_cmd` の両方を `codd_gate_regression.py` が保存するようにも読める。実際に同ツールが扱うのは `regression_cmd` だけで、`intake_cmd` と acceptance の生成・注入は対象外である。この非対称を明記した方がよい。

3. **検出と永続化の接続**

   モジュール表 289 行は、`codd_gate_regression.py` が `detect_wiring()` を生成可否の判断に使うとしている。永続化説明 309–314 行も、version、schema、capability 不足なら書かないとしている。一方、現物の生成 CLI は `detect_status()` を呼ぶだけであり、これは実体確認しか行わない。version、schema、capability は検査されない。設計を正とするなら生成 CLI は `detect_wiring()` 相当の完全な判定を受ける必要がある。現物を正とするなら設計書から完全検査の記述を落とす必要がある。

4. **doctor の所有者**

   §4.1 は `codd_gate_wiring.py` の所見を doctor 表示の材料とする一方、パッケージから sibling を import しないとも定める。パッケージ内 doctor が直接読む案は境界違反になる。外部 sibling CLI として明示実行するのか、install 時だけ使うのか、doctor 表示を非目標にするのかが未確定。

5. **任意パーサの接続**

   `codd_gate_debt.py` を任意 sibling としながら、設計書 294–295 行は `run_intake` が正規化結果を直接受ける前提で書いている。パッケージから import しない境界を守るなら、E3 の共通 task schema をパッケージ自身が汎用処理するか、外部コマンド側が完全な task spec を返す形に固定する必要がある。

### 現物コードが規範を満たしていない点

- `agent_project/configfile.py:201–229,376` は `_apply_codd_gate_auto_wiring()` を起動時に呼び、未設定の `regression_cmd` と `intake_cmd` をメモリ上で補完する。§4.1 の「明示設定だけ」「Config 生成で差し込まない」に反する。
- `agent_project/doctor.py:287–318,528` は `codd_gate_wiring` を import する。
- `agent_project/model.py:494–552` は `codd_gate_debt` を import する。
- このため sibling を削除すると、自動配線、doctor 所見、intake のレコード検証が変わる。「削除しても同一挙動」という責務は現 HEAD では成立しない。
- `codd_gate_regression.py:180` は完全な `detect_wiring()` ではなく、実体確認だけの `detect_status()` を使う。設計書が保証するとした version、schema、capability による書き込み抑止は現状ない。
- 設計書 309–312 行は CLI の処理を `build_regression_cmd()` → `upsert_config_text()` → `apply_to_file()` とするが、現物の `main()` は `apply_to_file()` を呼ばず、同等の書き込みを直接行う。結果は同じでも、関数の所有関係を正確にするなら記述か実装のどちらかを合わせる必要がある。

## 検証内容と結果

- 対象 worktree に `.codegraph/` は無かったため、設計書、依存成果 t3、関連モジュール、呼び出し元を `rg`、`git grep`、行番号付き表示で突き合わせた。
- `python3 -m unittest` で `test_codd_gate_detect.py`、`test_codd_gate_wiring.py`、`test_codd_gate_regression.py`、`test_codd_gate_routing.py` を実行し、71 件すべて PASS した。これは各部品の現行挙動を確認するもので、規範との一致を保証するテストではない。
- 境界の否定検索 `git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project` は `configfile.py`、`doctor.py`、`model.py` に一致した。したがって §4/§4.1 の境界は現 HEAD で未達。
- `git status --short --branch` は `## HEAD (no branch)`。リポジトリ配下は変更していない。

## 前提・未解決事項・範囲外

- 完了条件は、§4.1 の4観点について規範記述、現物との不一致、設計書で追加すべき説明を後続担当が判断できる粒度で整理すること、と解釈した。設計書や実装を直すことは本タスクに含めていない。
- 依存成果 t3 の E1–E3 境界整理を前提にし、本報告では §4.1 の値生成、有効化、永続化へ範囲を絞った。
- 「一意の書き手」は、手編集まで一人に限定する意味ではなく、「自動生成して YAML を更新するプログラムが `codd_gate_regression.py` 一つ」という意味に解釈した。
- agent-project、dashboard、テストは変更していない。

@followup 設計書の有効化・永続化を値別の表に直し、acceptance の保存先、intake_cmd に生成ツールがないこと、生成 CLI が行う互換検査の範囲を明記する。

@followup 実装側で `configfile.py` の起動時自動配線と、`doctor.py`/`model.py` から sibling への import を除去し、生成 CLI を `detect_wiring()` 相当の完全判定へ合わせるか、設計保証を実体確認だけへ縮めるか決める。

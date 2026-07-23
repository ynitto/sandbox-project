# codd-gate 連携境界の統合結論

## 統一境界

1. `agent_project/*` は E1〜E6 の汎用フックだけを提供する。codd-gate を名指し、import、探索、自動配線せず、未設定値は空のまま扱う。
2. `tools/agent-project/codd_gate_*.py` は任意の sibling 部品である。パッケージから直接結合せず、削除しても `agent_project` の挙動を変えない。
3. E2 `regression_cmd` と E3 `intake_cmd` の有効化経路は YAML または CLI だけとする。E1 はこの設定経路に含めず、task verify は task spec、acceptance は charter の `## acceptance` に書く。
4. YAML を機械的に永続化する sibling は `codd_gate_regression.py` だけで、書く値も `regression_cmd` 1 行だけとする。`intake_cmd` の生成・永続化ツールはなく、人か install 手順が YAML/CLI に設定する。`codd_gate_regression.py` は人か install 手順が明示実行し、起動時には走らない。

この境界は目標状態であり、現 HEAD では未達である。`configfile.py`、`doctor.py`、`model.py` に専用結合が計 14 行残っている。文書を直しても実装整理の完了とは判定しない。

## 文書別の変更

### `docs/designs/codd-gate-design.md`

- §4 の「値を書く主体」を E1 と E2/E3 に分ける。task verify と charter acceptance の保存先を明記し、YAML/CLI の対象を `regression_cmd` と `intake_cmd` に限定する。
- §4 の永続化説明を `codd_gate_regression.py` が `regression_cmd` 1 行だけを冪等注入する記述へ直す。`intake_cmd` を同ツールが書くように読める総則を削る。
- §4.1 冒頭の「表①〜③の文字列を yaml へ永続化」を、永続化対象は表①だけと分かる記述へ直す。表②と④は E1 の task/charter、表③は人か install 手順による YAML/CLI 設定とする。
- §4.1 の有効化節から acceptance を YAML/CLI の対象としている記述を外す。
- 生成前検査の保証は現行 `codd_gate_regression.py` に合わせ、「実体確認だけ」に縮める。同 CLI は `detect_status()` を呼び、version、schema、capability を実測する `detect_wiring()` を呼ばない。完全検査を保証する記述は、実装を接続する別タスクが完了するまで書かない。
- §4.2 の正典ゲートを README と同じ次の 1 コマンドへ統一する。パッケージだけを対象にするため、許可された sibling は一致しない。

  ```bash
  ! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
  ```

- §4.2 の「単一パターンだけを正典とし、厳格版とは連結しない」という説明を削る。統一後の 1 コマンドは自動配線、名指し、import の禁止をまとめて検査する。`_apply_codd_gate` は `_codd_gate` に包含されるが、自動配線の禁止を受入条件上で明示するため残す。
- §4 と §4.1 に「目標状態。現 HEAD は未達」と明記し、文書更新だけで整理済みに見えないようにする。

### `tools/agent-project/README.md`

- 現在の 1 コマンドを正典ゲートとして維持する。設計書側をこのコマンドへ合わせる。
- 「有効化は設定だけ」の対象を E2/E3 に限定し、E1 task verify と charter acceptance はそれぞれ task spec と charter に書く、と一文で区別する。
- `codd_gate_regression.py` が永続化するのは `regression_cmd` 1 行だけで、`intake_cmd` は永続化しない、という現行記述を維持する。
- sibling の検出説明を一括りにしない。`codd_gate_wiring.py` は実体、version、schema、capability を検査できる一方、現行 `codd_gate_regression.py` の生成 CLI は実体だけを確認する、と書き分ける。
- 境界は目標状態であり、上記ゲートが PASS するまでは未達、と明記する。

## 根拠の統合

- gate1 の独立検算と t7 は、E1 の保存先、`intake_cmd` の永続化主体、生成 CLI の検査範囲、正典ゲートの差を同じ箇所で確認している。
- t8 の 3 ファイル 14 行という集計と負条件の形は採用する。ただし「正典 §4 と README のコマンドが一致する」という判定は現物と矛盾するため採用しない。現物の設計書は `_apply_codd_gate` だけを `tools/agent-project` 全体で調べ、README は結合パターンを `agent_project` に限定している。
- gate1 自体は `verify=fail` であり、通過済みという前提を裏付ける入力はない。本成果物は fail の原因を上の統一判断で解消するための編集方針である。後続は文書を変更後、統一ゲートと文書差分の再検証を行う必要がある。

## 範囲外

agent_project、dashboard、テストの変更は行わない。

@followup 別タスクで `configfile.py`、`doctor.py`、`model.py` の専用結合と関連テストを汎用フック境界へ合わせる。

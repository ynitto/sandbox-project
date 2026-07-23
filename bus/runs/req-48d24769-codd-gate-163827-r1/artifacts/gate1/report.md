# verify 結果

verify=fail

## 判定

t3〜t8 の主要なコード調査は再現できたが、正典と README には実装判断を分岐させる矛盾が残っている。
また、t8 は正典と README の負の完了条件が一致すると報告しており、t7 および現物と食い違う。
この状態では「パッケージは汎用フックのみ・codd_gate_* は任意 sibling」という目標境界を一意に判定できない。

## 独立検算

- E1 の task verify と charter acceptance、E2 の `regression_cmd`、E3 の `intake_cmd` の実行部は汎用フックとして存在する。
- パッケージ内には専用結合が残る。`configfile.py` は起動時に自動配線し、`doctor.py` は
  `codd_gate_wiring`、`model.py` は `codd_gate_debt` を import する。sibling を削除すると設定値、
  doctor 所見、intake の不正レコード処理が変わるため、現行コードは目標境界に未到達。
- 結合 grep は3ファイル、重複を除く14行。個別には `_apply_codd_gate` 2行、`_codd_gate` 10行、
  `import codd_gate` 4行で、重複があるため単純合計18行にはならない。t8 の集計値は正しい。
- 正典ゲート `! git grep _apply_codd_gate -- tools/agent-project` と README ゲート
  `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  は、現 HEAD ではどちらも FAIL。
- sibling 関連の unittest 81件は PASS。これは現行挙動の確認であり、設計境界への適合を示すものではない。
- t3〜t8 の作業によるリポジトリ内変更はなく、検証時の `git status --short` も空。調査タスクに起因する
  スコープ外差分はない。

## issues

1. `docs/designs/codd-gate-design.md:301-304` は `regression_cmd`、`intake_cmd`、acceptance をまとめて
   yaml/CLI に書くとしている。E1 acceptance の保存先は charter の `## acceptance` で、対応する YAML キーや
   CLI はない。値別に保存先を分け、task verify は task spec、acceptance は charter、
   E2/E3 だけを YAML/CLI と明記すること。
2. `docs/designs/codd-gate-design.md:256-259` は `regression_cmd` と `intake_cmd` の機械的な書き手を
   `codd_gate_regression.py` にまとめたように読めるが、同書306-314行と
   `tools/agent-project/codd_gate_regression.py` は `regression_cmd` 1行しか永続化しない。
   `intake_cmd` には生成ツールがなく、人か install 手順が直接書く、と総則側にも明記すること。
3. `docs/designs/codd-gate-design.md:289,309-314` と `tools/agent-project/README.md:288-294` は、
   生成前に実体、version、schema、capability を検査すると読める。しかし
   `tools/agent-project/codd_gate_regression.py:180` は実体だけを見る `detect_status()` を呼び、
   `detect_wiring()` を使わない。生成 CLI を完全検査へ接続するか、設計書と README の保証を実体確認だけへ
   縮めること。目標境界を固定する設計タスクでは、どちらを正とするか明記が必要。
4. `docs/designs/codd-gate-design.md:361` の正典ゲートと
   `tools/agent-project/README.md:300` の必須ゲートは、パターンも対象パスも異なる。
   t7 はこの差を正しく報告したが、t8 の「正典 §4 と README の対象パスとコマンドが一致」は誤り。
   正典の単一パターンを採るか、名指し/import も含む README 版を正典へ昇格するか決め、両文書と t8 報告を
   同じ1コマンドへ直すこと。
5. `docs/designs/codd-gate-design.md:248-259,322-327` と README 284-294行が宣言する境界は、
   `agent_project/configfile.py:201-229,376`、`doctor.py:287-318,528`、
   `model.py:494-552` の現行コードでは成立しない。実装変更は今回の範囲外なので文書に「目標状態であり、
   現 HEAD は未達」と明記し、後続実装の受入として統一後の負条件を置くこと。現状を既成事実として読める
   文面のままでは整理完了を誤判定する。

{"ok": false, "issues": ["E1 acceptance の保存先を YAML/CLI とする設計書記述が実フックと矛盾", "intake_cmd の永続化主体が設計書内で不一致", "生成前の version/schema/capability 検査保証が現行 CLI と不一致", "正典と README の負の grep 条件が不一致で t8 の一致判定も誤り", "任意 sibling・非依存の目標境界は現行パッケージで未達"]}

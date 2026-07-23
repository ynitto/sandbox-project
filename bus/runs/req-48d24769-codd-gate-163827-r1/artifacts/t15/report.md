切り口: 有効化の判定根拠を「保存された設定値」に固定し、起動時検出では状態が変わらないことまで明示する。

## 成果候補

`docs/designs/codd-gate-design.md` §4 と §4.1 は、現 HEAD のままで完了条件を満たしている。

- §4 は、パッケージが codd-gate を名指し、import、自動配線しないと規定している。
- §4.1「有効化（値をどう入れるか）」は、有効化には値を yaml か CLI に書く必要があり、未設定なら連携なしで通過すると規定している。
- 同節は、Config 生成が値を自動注入せず、コマンド起動時の自動配線もないと規定している。

同じ内容の重複追記はしていない。また、対象文書は許可された書込範囲 `tools/agent-project/` の外にあるため、作業ツリーを変更していない。

## 検証内容と結果

- `git grep` で §4 の「名指し・import・自動配線しない」、§4.1 の「yaml か CLI に書く」「コマンド起動のたびに走る自動配線は無い」を確認した。
- `git diff --check`、`git diff -- docs/designs/codd-gate-design.md tools/agent-project`、`git status --short --untracked-files=all` はいずれも出力なしだった。
- 文書の現状確認だけなので、テストとリンタは実行していない。
- 設計上の受入 `! git grep _apply_codd_gate -- tools/agent-project` は失敗する。現 HEAD の `tools/agent-project/agent_project/configfile.py` に `_apply_codd_gate_auto_wiring` の定義と呼び出しが残っている。

## 前提・未解決事項・範囲外

- 完了条件は、設計書だけを読めば「有効化は yaml または CLI の明示設定に限る」「パッケージ内の名指し結合と暗黙の自動配線は禁止」の二点を判断できることとした。
- 現 HEAD の設計書はこの条件を満たすため、重複追記は不要と判断した。
- `_apply_codd_gate_auto_wiring` の撤去、agent-project、dashboard、テストの変更は本タスクの範囲外であり、手を加えていない。

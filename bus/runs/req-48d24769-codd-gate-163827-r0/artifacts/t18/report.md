# t18 成果報告

## 成果

- `tools/agent-project/README.md` に、`agent_project` は汎用フックだけを提供すること、`codd_gate_*.py` は任意 sibling であること、有効化は YAML/CLI 設定だけで行うことを明記した。
- `codd_gate_regression.py` が永続化する値を `regression_cmd` だけに限定し、`intake_cmd` と acceptance は書かないと明記した。
- 次の否定検索を任意例ではなく必須完了条件として README に追加した。

  ```bash
  ! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
  ```

## 検証

- `git diff --check`: PASS
- 変更ファイル確認: `tools/agent-project/README.md` だけ。agent_project と dashboard の実装・テストは未変更。
- 境界文言の検索: README への4条件と必須完了条件の反映を確認。
- 必須否定検索: FAIL。現行実装には `agent_project/configfile.py`、`doctor.py`、`model.py` の codd-gate 自動配線・import が残っている。実装変更は明示的にスコープ外なので修正していない。

## 前提・未解決事項

- 「変更してよいのは `tools/agent-project` 配下のみ」を書込範囲の最優先条件と解釈した。
- `docs/designs/codd-gate-design.md` はその範囲外に実在するため未変更。現状の §4 と §4.1 は4条件をほぼ記載済みだが、§4.2 は `_apply_codd_gate` 単独検索を正典の完了条件とし、指定された連結パターンを任意の厳格例に留めている。
- @followup `docs/designs/codd-gate-design.md` を所有できるタスクへ割り当て、§4.2 の正典を上記の連結パターンへ一本化する。README の変更と同じコミットへ含める必要があるなら、本タスクの許可範囲を `docs/designs/codd-gate-design.md` に拡張する。

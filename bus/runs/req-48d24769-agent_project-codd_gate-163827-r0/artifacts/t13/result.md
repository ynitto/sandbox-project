verify=pass

- 指定コマンドは `tools/agent-project/tests/test_agent_project.py` が存在しないため exit 2。
- 実在する移動先を `PYTHONPATH=tools/agent-project python3 -m unittest -v tests.test_backlog.TestIntake.test_run_intake_enqueues_and_dedups_by_id` で実行し、`Ran 1 test` / `OK`。
- テストは2件の JSON 配列を取り込み、再実行が空になり、backlog が `I1` と `I2` の2件だけであることを検証している。
- `run_intake` は `_parse_intake_records` で JSON を object/array に正規化し、既存 backlog の stem と照合して同一 id を再投入しない。
- worktree は clean。`main...HEAD` の差分はすべて `tools/agent-project` 配下で、`git diff --check` 成功。
- (minor) synth2 の「7ファイル」と異なり、独立に確認した `git diff --name-only main...HEAD` は6ファイル。

{"ok": true, "issues": ["(minor) synth2/result.md は変更範囲を7ファイルと報告しているが、現 worktree の git diff --name-only main...HEAD は tools/agent-project 配下6ファイル。成果物の件数表記を現状に合わせて確認・修正するとよい。"]}

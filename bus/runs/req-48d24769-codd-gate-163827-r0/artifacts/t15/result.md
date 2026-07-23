verify=fail

正典 `docs/designs/codd-gate-design.md` §4・§4.1 と `tools/agent-project/README.md` の一貫性ゲート節を独立に照合した。

- E2 `regression_cmd`、E3 `intake_cmd`、E1 acceptance の各サブコマンドと役割、任意 sibling、自動検出、有効化、未検出時 no-op は一致。
- README 274行目の「結合は共通スキーマのみ」は、正典 258–259行目の「共通データ契約と汎用フックへ置く文字列値」に反する。README を「結合は共通スキーマと E1–E3 の汎用フックへ置くコマンド文字列のみ」などへ修正する。
- README 280–281行目は E1 の acceptance だけを記載し、正典 266行目の E1 タスク verify (`codd-gate check ...`) を欠く。E1 の二つの意味を揃えるため、修復タスクの状態アサーションを追記する。
- README 283–285行目は `codd_gate_regression.py` が `regression_cmd`/`intake_cmd` の双方を永続化するように読めるが、正典 306–312行目では同 CLI が永続化するのは `regression_cmd` 1行だけ。README で永続化対象を明記し、`intake_cmd` は手書きであることを分離する。
- README 284行目の `python3 codd_gate_regression.py ...` はリポジトリルートから実行不能。正典 308行目および実在パスに合わせて `python3 tools/agent-project/codd_gate_regression.py ...` とする。

差分確認: t14 は README 1ファイルのみ、`git diff --check` PASS。正典 §4.2 の境界ゲートは現実装に `_apply_codd_gate_auto_wiring` が残るため現時点では FAIL だが、実装除去は本タスクの明示的スコープ外なので今回の文書整合性判定の追加 fail 理由にはしていない。

{"ok": false, "issues": ["tools/agent-project/README.md:274 の『結合は共通スキーマのみ』を、docs/designs/codd-gate-design.md:258-259 に合わせて『共通スキーマと E1-E3 の汎用フックへ置くコマンド文字列のみ』へ修正する。", "tools/agent-project/README.md:280-281 に、docs/designs/codd-gate-design.md:266 の E1 タスク verify（codd-gate check による状態アサーション）を追記する。", "tools/agent-project/README.md:283-285 は codd_gate_regression.py の永続化対象が regression_cmd 1行だけであることを明記し、intake_cmd の手書き有効化と分離する。", "tools/agent-project/README.md:284 の実行例を python3 tools/agent-project/codd_gate_regression.py --config .agent/agent-project.yaml に修正する。"]}

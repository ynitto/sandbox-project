# 検証結果

verify=fail

## 独立検算

- `main...HEAD` の変更は `docs/designs/codd-gate-design.md` と `tools/agent-project/README.md` の2文書だけ。
- 差分は設計書 `+87/-30`、README `+12/-9`、合計 `+99/-39` で `git diff --stat` と一致。
- `agent_project`、dashboard、テストの変更は0件。作業ツリーは clean。
- `git diff --check main...HEAD` は PASS。
- §4 は E1（verify/acceptance）、E2（regression_cmd）、E3（intake_cmd）と「パッケージは汎用フックのみ／`codd_gate_*.py` は任意 sibling」を明記し、README も同じ目標境界を参照している。

## issues

1. `docs/designs/codd-gate-design.md:354` — 正典の必須完了条件 `! git grep _apply_codd_gate -- tools/agent-project` は自動配線関数だけを検出し、`agent_project/doctor.py` と `agent_project/model.py` に `import codd_gate_*` が残っても PASS する。§4 が禁じる「名指し・import・自動配線」のうち import を見逃す false green なので、現在「より厳しく」と任意扱いしている package-scoped grep も必須完了条件にするか、両コマンドを必須として明記すること。
2. `tools/agent-project/README.md:279` — 「import・配線層は無い」「sibling を削除しても同一挙動」と現行仕様として断定しているが、現在は `configfile.py` の `_apply_codd_gate_auto_wiring`、`doctor.py` / `model.py` の sibling import が存在する。設計書 §4.2 は除去を後続実装としているため、README は目標状態・移行前であることを明記し、実装完了後に現行仕様の記述へ切り替えること。
3. (minor) `docs/designs/codd-gate-design.md:256`、`tools/agent-project/README.md:280` — `codd_gate_regression.py` が `regression_cmd` と `intake_cmd` の双方を永続化するようにも読めるが、現実装は `regression_cmd` のみ。`intake_cmd` は手動設定と明記すること。

現在の受入 grep が非PASSなのは後続実装前なので、それ自体は今回の失敗理由ではない。失敗理由は、文書で固定する完了条件が規範全体を検証できないことと、README が移行前の実装状態と矛盾すること。

{"ok": false, "issues": ["docs/designs/codd-gate-design.md:354: 必須 grep が package 内の codd_gate import を見逃して false green になるため、package-scoped grep も必須化する", "tools/agent-project/README.md:279: 後続実装で実現する目標状態を現行挙動として断定しているため、移行前であることを明記する", "(minor) docs/designs/codd-gate-design.md:256 / tools/agent-project/README.md:280: codd_gate_regression.py が永続化するのは regression_cmd のみと明記する"]}

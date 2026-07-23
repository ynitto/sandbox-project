# gate2 検証報告

verify=fail

## 判定

`agent-reviewer` の `document` perspective と独立検算を突き合わせ、重大問題を 2 件確認した。

1. **負の grep が自己一致し、かつ規範境界を網羅しない**
   - 場所: `tools/agent-project/README.md:308`、`docs/designs/codd-gate-design.md:354-386`
   - 現行条件 `! git grep _apply_codd_gate -- tools/agent-project` は、検索対象内の README に書かれた
     コマンド自身へ一致する。現 HEAD で exit 1 であり、実装から
     `_apply_codd_gate_auto_wiring` を削除しても README が残る限り PASS しない。
   - また `_apply_codd_gate` だけでは、`agent_project/doctor.py` と `model.py` に残る
     `_codd_gate_*` / `import codd_gate_*` を検出できず、§4 の「名指し・import・自動配線しない」を
     完了条件として固定できない。
   - 修正: t19 草稿のパッケージ限定条件
     `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
     を正典 §4.2 と README の双方へ反映する。これなら sibling 部品・tests・README を誤検出しない。

2. **永続化前の自動検出記述が現行コードと不一致**
   - 場所: `tools/agent-project/README.md:292-301`、
     `docs/designs/codd-gate-design.md:290,309-314`、
     `tools/agent-project/codd_gate_regression.py:180`、
     `tools/agent-project/codd_gate_status.py:119-138`
   - 文書は `codd_gate_regression.py` が実体、version、schema、capability を検査し、非互換なら
     書かないと断定する。実装は `detect_status()` を呼ぶだけで、同関数は実体だけを確認し、
     version/schema は適合扱い、capability は未検査である。
   - 修正: 今回は実装変更禁止なので、現状を「実体検出のみ」と明記し、version/schema/capability
     を永続化前に検査する実装は `@followup` に分離する。将来設計として残す場合は未実装であることと
     `detect_wiring()` 統合の受入条件を明示する。

## 合格した観点

- E1（task verify / acceptance）、E2（done 前 `regression_cmd`）、E3（周期的 `intake_cmd`）の配置は整合。
- `codd_gate_regression.py` が永続化するキーは `regression_cmd` だけで、`intake_cmd` は書かない。
- この run のリポジトリ変更は `tools/agent-project/README.md` だけ。`agent_project/`、dashboard、tests は非変更。
  `docs/designs/codd-gate-design.md` の main 比差分は別 run の既存コミットであり、t19 は書込制約を守って未反映。
- `git diff --check`（t18 commit）PASS。
- `python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate*.py'`: 81 tests PASS。
- worktree は clean。

{"ok": false, "issues": ["tools/agent-project/README.md:308 と docs/designs/codd-gate-design.md §4.2 の負の grep は README 自身へ一致して常に失敗し、さらに doctor.py/model.py の名指し・import を検出しない。t19 のパッケージ限定・統合パターンを両文書へ反映すること", "tools/agent-project/README.md:292-301 と docs/designs/codd-gate-design.md §4.1 は永続化 CLI が version/schema/capability を検査すると記すが、codd_gate_regression.py:180 が呼ぶ detect_status() は実体しか検査しない。現状記述へ直し、完全検査の実装は @followup に分離すること"]}

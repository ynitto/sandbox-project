# adversarial verification gate

判定: **FAIL**

指定コマンドは exit 0、`git diff --check main...HEAD` も PASS。しかし、正典整合とスコープ適合が欠けるため通過不可。

## 依存案比較

| dep | 要求適合 | 正確さ・根拠 | 完成度 | 評価 |
|---|---|---|---|---|
| t15 | README の具体的不整合を最も細かく指摘 | 実行パス、E1 task verify、`regression_cmd` だけを永続化する点は現物と一致 | ブランチ全体の `docs/designs/**` スコープ違反と、§4.2 の false green を見落とす | 次点 |
| t16 | 正典の機械的完了条件が要求全体を検証しない核心を指摘 | `_apply_codd_gate` だけでは `doctor.py` / `model.py` の import が残っても PASS するとの指摘は再現済み | 指定コマンドの exit 0 と書込範囲違反を欠く | **最良** |
| t17 | 指定コマンド成功とブランチ全体のスコープ違反を正確に検出 | exit 0、変更2ファイル、clean、diff-check PASS は再現済み | §4.2 を整合済みとした点が元要求の `_apply_codd_gate|_codd_gate|import codd_gate` 必須化に反する | 不採用 |

## 統合所見

- 正典整合: **FAIL**。`docs/designs/codd-gate-design.md` §4.2 は `_apply_codd_gate` の不在だけを必須とし、`_codd_gate` / `import codd_gate` の不在を任意扱いしている。元要求と§4の「名指し・import・自動配線しない」を完全には固定できず false green になる。
- スコープ適合: **FAIL**。`main...HEAD` は `docs/designs/codd-gate-design.md` と `tools/agent-project/README.md` を変更しており、前者は許可された `tools/agent-project/**` 外。
- 指定コマンド: **PASS**。原文どおり再実行して exit 0。これは語句の存在確認であり、上記の正典・所有範囲違反を打ち消さない。

## 修正対象（1ファイル・1観点）

1. `docs/designs/codd-gate-design.md` — **完了ゲート**: package 限定の `! git grep -nE '_codd_gate|import codd_gate' -- tools/agent-project/agent_project` を任意例ではなく必須条件にする。
2. `docs/designs/codd-gate-design.md` — **所有範囲**: この変更を `docs/designs/**` の書込権限を持つ専用タスクへ移すか、明示的に ownership を拡張する。
3. `tools/agent-project/README.md` — **正典との結合境界**: 「共通スキーマのみ」を、共通スキーマと E1–E3 の汎用フックに置くコマンド文字列のみ、へ揃える。
4. `tools/agent-project/README.md` — **永続化**: `codd_gate_regression.py` が注入するのは `regression_cmd` 1行だけで、`intake_cmd` は手動設定であると分離する。
5. `tools/agent-project/README.md` — **実行例**: リポジトリルート基準で `python3 tools/agent-project/codd_gate_regression.py ...` とする。
6. `tools/agent-project/README.md` — **E1 完全性**: acceptance に加え、修復タスクの `codd-gate check ...` 状態アサーションを記載する。
7. `tools/agent-project/README.md` — **時制**: import・自動配線なし／sibling 削除時も同一挙動は後続実装後の目標状態だと明示する。

`@followup docs-design-owner docs/designs/codd-gate-design.md §4.2 の必須ゲートを package 限定の厳密 grep に修正し、正典変更を所有範囲内で取り込む。`

{"winner": "t16"}

# codd-gate-docverify-22 独立検証

総合判定: **FAIL**

`docs/designs/codd-gate-design.md` §4.2 が強化版 grep を必須条件にしていないため、主要求を満たさない。リポジトリは変更していない。

## 検証結果

| 項目 | 判定 | 結果 |
|---|---|---|
| 元要求の連結シェルコマンド | PASS | 原文どおり実行し exit 0。README と設計書の要求語句を検出した |
| `git diff --check` | PASS | 作業ツリー対象、`main...HEAD` 対象とも exit 0 |
| §4.2 の必須完了条件 | FAIL | `docs/designs/codd-gate-design.md:361` は旧条件 `! git grep _apply_codd_gate -- tools/agent-project` のまま |
| 強化版の否定 grep | FAIL | `! git grep -nE '_apply_codd_gate\|_codd_gate\|import codd_gate' -- tools/agent-project/agent_project` は exit 1。14件残存 |
| §4 の E1 | PASS | acceptance の負債ラチェットと修復タスクの `codd-gate check` を定義。README も両方を記載 |
| §4 の E2 | PASS | `regression_cmd` による毎タスクの差分ゲートで一致 |
| §4 の E3 | PASS | `intake_cmd` による負債タスクの周期的・冪等取り込みで一致 |
| §4.1 の自動検出境界 | PASS | sibling を人または install 手順が明示起動した場合だけ検出し、パッケージは探索・import しない規範で一致 |
| §4.1 の有効化境界 | PASS | YAML/CLI に人または install 手順が値を書く場合だけ有効化し、未設定は連携なしで一致 |
| §4.1 の永続化境界 | PASS | `codd_gate_regression.py` だけが YAML の `regression_cmd` 1行を冪等注入し、`intake_cmd` は人または install 手順が設定する規範で一致 |
| 差分ファイル | PASS | `main...HEAD` は `docs/designs/codd-gate-design.md` と `tools/agent-project/README.md` の2件だけ |
| 実装・dashboard・テストの無変更 | PASS | `tools/agent-project/agent_project/**`、`tools/agent-dashboard/**`、テストに差分なし |
| 作業ツリー | PASS | `git status --short` は出力なし |

## 不一致

`docs/designs/codd-gate-design.md:356-385` は、§4 の「パッケージから codd-gate を名指し・import・自動配線しない」を規範にしている。一方、必須条件は `_apply_codd_gate` だけを検査し、強化版を「より厳しく見たいなら」と任意扱いする。この条件では `_apply_codd_gate_auto_wiring` だけを削除した後も、`_codd_gate*` や `import codd_gate_*` が残った状態を PASS にできる。

強化版が検出した14件は次のとおり。

- `agent_project/configfile.py`: 201、220、376行
- `agent_project/doctor.py`: 287、290、295、303、309、314、528行
- `agent_project/model.py`: 494、504、512、552行

README 285–291行は名指し、import、自動配線の不在を明記しているため、本文の境界は設計書 §4・§4.1 と一致する。ずれているのは設計書 §4.2 の機械的な必須条件である。

独立文書レビューも `REQUEST_CHANGES`。同じ false green を Critical 1件と判定した。追加の軽微な指摘として、設計書 289行の `doctor_findings()` はパッケージ側 doctor 連携の除去後に誰が呼ぶのかが不明である。

## 前提と未解決事項

- 差分範囲は、作業ブランチ全体を表す `main...HEAD` で判定した。未コミット差分だけを見る `git diff` は空である。
- `codd-gate-docfix-21` は変更なしと報告されているため、現在の HEAD をその後の検証対象とした。
- 強化版 grep の exit 1 は現行実装の残存を示す。実装整理は明示された範囲外なので修正していない。
- 未解決事項は、§4.2 の必須条件を package 限定の強化版へ置き換えること。これは `docs/designs/**` の書込権限を持つ担当が行う必要がある。

@followup docs-design-owner `docs/designs/codd-gate-design.md` §4.2 の必須条件を `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project` 相当へ置き換え、旧条件と任意扱いの説明を削除してください。

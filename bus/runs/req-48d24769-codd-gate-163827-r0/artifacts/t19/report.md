# t19 検証報告

## サマリー

**FAIL（pass 不可）**。

元要求の連結シェルコマンドと `git diff --check` は exit 0 で、差分範囲と成果コミットの再現性も適合した。
しかし、正典 `docs/designs/codd-gate-design.md` §4.2 の必須ゲートは現行 target でも
`! git grep _apply_codd_gate -- tools/agent-project` のままであり、`_codd_gate` と
`import codd_gate` を必須検査に含めていない。t18 の `codd-gate-design.patch` は target に未適用である。
必須項目が1件欠けるため、総合判定を pass にしない。

## 検証内容と結果

| 項目 | 結果 | 根拠 |
|---|---|---|
| 元要求の連結シェルコマンド | PASS | 指定された4段の `grep` / `test -f` をそのまま連結して実行し `LINKED_EXIT=0` |
| `git diff --check` | PASS | 作業ツリー対象で exit 0。併せて `git diff --check main...HEAD` も exit 0 |
| 正典 §4.2 の必須 grep | **FAIL** | 必須と記されたコードブロックは `_apply_codd_gate` のみ。3パターンの連結 grep は存在せず、後段の `_codd_gate\|import codd_gate` は「より厳しく見たいなら」という任意例 |
| 強化ゲートの検出能力 | PASS（参考） | `! git grep -nE '_apply_codd_gate\|_codd_gate\|import codd_gate' -- tools/agent-project/agent_project` は14件を検出して exit 1。現行実装の自動配線・private 名・固有 import を漏らさない |
| 変更範囲 | PASS | `git diff --name-only main...HEAD` は `docs/designs/codd-gate-design.md` と `tools/agent-project/README.md` の2文書だけ |
| 実装・dashboard・テスト差分なし | PASS | `agent_project/`、`tools/agent-dashboard/`、`**/test/**`、`**/tests/**` を pathspec で照合し差分0 |
| 成果コミットの実在 | PASS | `git cat-file -e abe5906a18149a1ab4a4c97654bfa7d7950eb763^{commit}` が exit 0 |
| 成果コミットの再現性 | PASS | `HEAD` と `refs/heads/ap/codd-gate-163827` はともに `abe5906a18149a1ab4a4c97654bfa7d7950eb763`。tracked 差分・index 差分・porcelain status はすべて空。tree は `57e302f155ea3ef2f7bd12e29a75eb56dce0bbb0` |

成果コミット `abe5906a18149a1ab4a4c97654bfa7d7950eb763` 自体の変更は
`tools/agent-project/README.md` のみである。target 全体では先行コミット由来の設計書変更を含むため2文書差分だが、
t18 が成果物として渡した §4.2 強化パッチの内容はコミットされていない。

## 採用した前提・未解決事項・範囲外

- 「現行 target」は指定 worktree の `HEAD`、比較基準は `main...HEAD` と解釈した。
- 「成果コミットが再現可能」は、commit object が存在し、target branch ref と HEAD がその SHA を指し、
  clean な作業ツリーが同じ commit 内容を再現していること、と解釈した。
- 強化した否定 grep の exit 1 は、将来境界の実装整理が未完了であることを正しく示す。実装変更は本タスクの範囲外であり、変更していない。
- リポジトリは一切変更していない。出力は本報告だけである。
- 未解決: `t18/codd-gate-design.patch` を正典へ反映し、§4.2 の必須ゲートを3パターン連結版へ置き換える必要がある。

@followup: 設計書の書込権限を持つタスクで `t18/codd-gate-design.patch` を target に反映し、同じ検証を再実行する。

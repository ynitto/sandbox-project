# t7 成果報告

## サマリー

`tools/agent-project/README.md` 272–301 行の「一貫性ゲート」を、正典
`docs/designs/codd-gate-design.md` §4、§4.1、§4.2 と照合した。

6観点のうち、`agent_project` 非依存と sibling の位置づけは一致する。自動検出も方針は一致するが、
設計書内で生成可否の判定経路が揺れている。明確な差異は `intake_cmd` の永続化、E1 の有効化先、
完了ゲートの3点。`regression_cmd` は永続化対象が1行だけという結論は一致するものの、互換検査の
説明が設計書内で一致していない。

## 差異一覧

| 観点 | README | 設計書 | 判定 |
|---|---|---|---|
| `agent_project` 非依存 | パッケージは汎用フックだけを持ち、codd-gate を名指し、import、自動配線しない（284–291行）。 | 同じ境界を規定する（§4、248–251行）。 | 散文は一致。ただし README の完了ゲートは設計書の正典ゲートと異なる。 |
| sibling | `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling。削除してもパッケージの挙動は変わらない（289–294行）。 | 配置、型への非依存、パッケージからの非 import、削除時の同一挙動を規定する（§4、252–255行、§4.1、322–327行）。 | 差異なし。README は設計書の要点を保っている。 |
| 自動検出 | 人か install 手順が sibling を明示起動したときだけ、実体、バージョン、schema、対応機能を検出する。パッケージは探索しない（288–294行）。 | 読み取り専用の sibling が同じ4項目を検出し、起動時の Config 生成では走らない（§4.1、284–290行、301–320行）。 | 方針は一致。ただし設計書 289行は `detect_wiring()` が生成可否を完全判定するとし、309行は `build_regression_cmd()` が detect/status/routing で判定すると書く。README は完全検査だけを記し、この経路差を表に出していない。 |
| `regression_cmd` | 人か install 手順が YAML/CLI で有効化する。`codd_gate_regression.py` を明示実行した場合だけ YAML の1行を永続化する（277–278行、285–294行）。 | §4.1 の具体記述は同じ（290行、306–314行）。一方、§4 の総則は `regression_cmd` と `intake_cmd` をまとめて生成ツールが注入するようにも読める（256–259行）。 | 結論は一致。設計書の総則だけが曖昧。生成前に capability まで検査するかは前項の経路差が残る。 |
| `intake_cmd` | 人か install 手順が設定し、`codd_gate_regression.py` は永続化しない（279–280行、285–287行）。 | §4 の総則と §4.1 冒頭は、表①〜③の値を `codd_gate_regression.py` が YAML へ永続化するように読める（256–259行、275–280行）。後段は同ツールが `regression_cmd` 1行だけを扱う（306–314行）。 | README は後段と一致するが、設計書の前段とは不一致。正典内で記述が衝突している。 |
| YAML/CLI 有効化 | YAML/CLI の対象を `regression_cmd` と `intake_cmd` に限定する。E1 task verify はタスク、E1 acceptance は charter に置く（281–286行）。 | 差し込み点表は E1 の正位置を task/charter とする（263–266行）が、§4.1 は acceptance も YAML/CLI に書くとしている（301–304行）。 | README の保存先が正確。設計書 §4.1 の「acceptance を yaml か CLI」は README および差し込み点表と不一致。acceptance 用の設定キーや CLI は示されていない。 |

## 完了ゲートの差異

README 300行は次の1本を必須ゲートにする。

```bash
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

設計書 §4.2 は、正典の完了条件を次の単一パターンに固定する。

```bash
! git grep _apply_codd_gate -- tools/agent-project
```

設計書は、名指しと import の不在を確認する厳格版を別コマンドとして示し、2つを連結しないと明記している
（371–381行）。README は `_apply_codd_gate` を包含する `_codd_gate` と `import codd_gate` を1本にまとめ、
パスをパッケージへ限定した。検査意図は設計書の厳格版に沿うが、「正典の完了条件」とコマンドが一致しない。

## 検証内容と結果

- worktree に `.codegraph/` は無かったため、README の対象節、設計書 §4〜§4.2、依存成果 t5 を行番号付きで照合した。
- README のゲート相当を実行すると、`configfile.py`、`doctor.py`、`model.py` に一致し、現 HEAD では FAIL。
- 設計書の正典ゲート相当も `configfile.py` の `_apply_codd_gate_auto_wiring` に一致し、現 HEAD では FAIL。
- `git status --short` は空。リポジトリ配下を変更していない。調査タスクのためテストは実行していない。

## 前提、未解決事項、範囲外

- 完了条件は、指定された6観点について README と正典設計書の記述差を列挙し、後続担当が修正文を決められること、と解釈した。
- 比較対象は文書同士であり、現物実装との差はゲート実行結果の確認に限った。t5 が報告した実装不一致の修正は本タスクに含めていない。
- 「自動検出」は通常起動時の自動配線ではなく、人か install 手順が sibling 生成ツールを明示起動した後、そのツール内部で行う自動判定、と解釈した。
- README と設計書のどちらを修正するかは未決。正典を優先するなら、README のゲートを正典の1行へ合わせる必要がある。一方、名指しと import まで必須にする意図なら、設計書 §4.2 側の「連結しない」という決定を変更する必要がある。
- agent-project、dashboard、テストは変更していない。

@followup 設計書 §4/§4.1 の永続化説明を `regression_cmd` 1行だけに限定し、`intake_cmd` には生成ツールがないこと、E1 は task/charter に書くことを明記する。

@followup README と設計書 §4.2 のどちらを正とするか決め、完了ゲートを1種類に統一する。

# t22 成果報告

## サマリー

rebase と競合解消は未実施。作業ツリーを変更せず停止した。

- 作業 HEAD: `916d87c31897e09ad1b1958b21934049803af05f`
- 最新ローカル `main`: `1d6a516f2e40c42ad2a7f8de5f70981b5a140e39`
- merge-base: `9a7302ffc170b047c504d0f9db0ec5581ccb3354`
- `git merge-tree main HEAD` は次の競合を検出した。
  - `docs/designs/codd-gate-design.md`
  - `tools/agent-project/README.md`

依頼内の git 利用規約が `rebase`、`checkout`、`branch`、`commit`、`push` を禁止し、agent-flow が
それらを行うと定めているため、ワーカーから rebase を開始できない。さらに変更許可は
`tools/agent-project/` 配下だけなので、競合する設計書正典も編集できない。synth2 と t20 の
`design-proposed-edit.md` は確認したが、正典へは反映していない。

`tools/agent-project/README.md` の現 HEAD には、synth2 の次の内容が既に入っている。

- `agent_project` は汎用フックだけを提供する。
- `codd_gate_*.py` は任意 sibling 部品で、パッケージから探索、import、直接結合しない。
- 永続化を行うのは、明示実行された `codd_gate_regression.py` だけ。
- yaml または対応 CLI へ値を明示設定した場合だけ有効になり、自動検出だけでは有効にならない。

## 検証

| 検証 | 結果 |
|---|---|
| `git status --short --branch` | `## HEAD (no branch)`、変更なし |
| `git diff --check` | PASS |
| `git diff --name-only` | 出力なし。実装、dashboard、テストを含め変更なし |
| `git merge-tree main HEAD \| rg '^(Auto-merging\|CONFLICT)'` | 上記 2 ファイルの content conflict を確認 |
| `! git grep -E '_apply_codd_gate\|_codd_gate\|import codd_gate' -- tools/agent-project` | FAIL、exit 1 |
| `! git grep -E '_apply_codd_gate\|_codd_gate\|import codd_gate' main -- tools/agent-project/agent_project` | PASS、exit 0 |

指定された広い負の条件は、許可されている sibling 部品、テストの import、README に記載した
コマンド自身にも一致する。したがって「`codd_gate_*.py` は sibling 任意部品として保持する」
「実装・テストは変更しない」と両立せず、rebase 後の `main` でも exit 1 になる。境界の機械判定として
実行可能なのは、検索先をパッケージ本体へ限定した後者である。

## 前提・未解決事項・範囲外

- 前提: 同一依頼内の明示的な git 利用規約と書込範囲を、タスク見出しの rebase 指示より強い操作制約として扱った。
- 未解決: agent-flow が最新 main へ rebase を開始し、設計書正典の書込権限を持つ工程で t20 案を反映する必要がある。
- 未解決: 完了条件は
  `-- tools/agent-project/agent_project` に限定するか、sibling、tests、README を除外する別の pathspec に直す必要がある。
- 範囲外で新たな実装上の問題は調査していない。

@followup agent-flow 管理下で rebase を開始し、t20 の文体修正版を設計書 §4〜§4.2 へ反映する。

@followup 負の完了条件の検索先を `tools/agent-project/agent_project` に限定する。

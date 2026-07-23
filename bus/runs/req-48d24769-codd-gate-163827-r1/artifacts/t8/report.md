# codd-gate パッケージ境界の負の完了条件

## 完了条件と前提

本タスクの完了条件を、指定された worktree の HEAD
`abe5906a18149a1ab4a4c97654bfa7d7950eb763` について、次の3点を確定することとした。

1. `tools/agent-project/agent_project` 内に残る
   `_apply_codd_gate`、`_codd_gate`、`import codd_gate` の全一致箇所を `git grep` で確認する。
2. 正典 `docs/designs/codd-gate-design.md` §4 の
   「パッケージは codd-gate を名指し・import・自動配線しない」に対応する対象パスとパターンを、
   設計上の負の完了条件として確定する。
3. 調査だけを行い、agent_project、dashboard、テスト、設計書は変更しない。

パターンは正典と `tools/agent-project/README.md` に記載済みの決定的ゲートをそのまま採用した。
`git grep` は追跡対象だけを検査するため、完了判定はコミット対象コードの境界検査と解釈した。

## 確定した負の完了条件

対象パス:

```text
tools/agent-project/agent_project
```

禁止パターン（拡張正規表現）:

```text
_apply_codd_gate|_codd_gate|import codd_gate
```

完了判定コマンド:

```bash
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

このコマンドが exit 0、すなわち `git grep` が不一致を返したときに境界整理を完了とする。
対象を `agent_project` パッケージに限定するため、許可された任意 sibling
`tools/agent-project/codd_gate_*.py` とその単体テストは検査対象外になる。

## 現在の残存箇所

現時点では計14行、3ファイルが一致し、負の完了条件は未達である。

- `tools/agent-project/agent_project/configfile.py`: 3行
  - `_apply_codd_gate_auto_wiring` の定義
  - `_codd_gate_wiring_module` の呼び出し
  - `_apply_codd_gate_auto_wiring` の呼び出し
- `tools/agent-project/agent_project/doctor.py`: 7行
  - `_codd_gate_wiring_module`、`doctor_codd_gate_findings` の定義・呼び出し・説明
  - `import codd_gate_wiring` 2箇所
- `tools/agent-project/agent_project/model.py`: 4行
  - `_codd_gate_debt_module` の定義・呼び出し
  - `import codd_gate_debt` 2箇所

一致行番号は `configfile.py:201,220,376`、`doctor.py:287,290,295,303,309,314,528`、
`model.py:494,504,512,552` である。

個別確認では `_apply_codd_gate` が2行、`_codd_gate` が10行、
`import codd_gate` が4行だった。前者2行は `_codd_gate` にも包含されるため、
3パターンの単純合計ではなく、結合パターンの一意な一致行数14を採用している。

## 検証結果

- 結合パターンの `git grep -nE` を実行し、上記14行を確認した。
- `git grep -lE` で一致ファイルが `configfile.py`、`doctor.py`、`model.py` の3件だけであることを確認した。
- 正典 §4 と `tools/agent-project/README.md` の記載を照合し、対象パスとコマンドが一致することを確認した。
- 調査前後の `git status --short` は空で、指定 worktree に変更はない。
- 調査のみのため、テスト・リンタ・型チェックは実行していない。

## 未解決事項・範囲外

- 負の完了条件を満たすには上記3ファイルから専用の自動配線、doctor 表示、debt 正規化結合を除く実装変更と、
  対応するテスト改修が必要だが、本タスクの範囲外なので変更していない。
- `_apply_codd_gate` は `_codd_gate` に包含されるため検索上は冗長だが、禁止対象の自動配線を受入条件上で明示する
  意図があると解釈し、正典どおり残した。

@followup 別タスクで上記3ファイルの codd-gate 専用結合を除去し、関連テストを汎用フック境界へ合わせる。

切り口: 「何を検出するか」より先に依存方向と欠落時の不変条件を固定し、汎用 hooks レイヤだけを自動検出の主体にする。

## 成果候補

`docs/designs/codd-gate-design.md` §4.1 の冒頭説明を、次の記述へ置き換える。

```markdown
### 4.1 汎用 hooks レイヤと任意 sibling 部品（`tools/agent-project/codd_gate_*.py`）

`codd_gate_*.py` は `tools/agent-project/` 直下に置く任意の sibling 部品であり、
`agent_project` パッケージの構成要素ではない。パッケージからモジュール名を列挙したり、
`import codd_gate_*` したり、codd-gate 固有の型や設定を参照したりしない。

任意部品の発見と呼び出しは、外部部品を名前に依存せず扱う汎用 hooks レイヤが担う。
hooks レイヤは共通の登録規約と入出力契約だけを知り、規約に合う sibling を起動時に自動検出する。
検出後も `agent_project` に渡すのは `regression_cmd`、`intake_cmd`、doctor finding など既存の汎用値だけである。
codd-gate の実在確認、能力判定、コマンド生成、出力の正規化は検出された sibling 側に閉じる。

この依存方向は一方通行とする。

```
agent_project → 汎用 hooks 契約 ← codd_gate_*.py → codd-gate
```

`codd_gate_*.py` が存在しない、読み込めない、または codd-gate が利用できない場合、hooks レイヤは
未登録として扱う。`agent_project` はエラーにせず、汎用フックが未設定の従来動作を続ける。
したがって sibling 一式を削除してもパッケージの import、設定解決、verify、intake、doctor は変わらない。
```

既存のモジュール一覧はこの境界説明の後へ残す。ただし、一覧中の
「`codd_gate_wiring.py` はパッケージへ配線しない」や、後続の
「パッケージは `codd_gate_*` を import しない」は上記と重複するため削る。

## 検証内容と結果

- 現物の §4 と §4.1 を確認し、E1〜E3 が `agent_project` の汎用値として定義されていることを確認した。
- `tools/agent-project/agent_project/configfile.py`、`doctor.py`、`model.py` と
  `codd_gate_detect.py`、`codd_gate_wiring.py` を突合した。現 HEAD にはパッケージから
  `codd_gate_*` を遅延 import する経路と `_apply_codd_gate_auto_wiring` が残っており、
  本候補は現状説明ではなく目標境界を定める記述である。
- 依存成果 t9 の E1〜E3 対照表と突合した。本候補は値ごとの有効化主体を繰り返さず、
  自動検出の主体、依存方向、欠落時の挙動を補う。
- `git status --short --untracked-files=all` で指定 worktree に変更がないことを確認した。
  文書候補のみのため、コードのテストやリンタは実行していない。

## 前提・未解決事項・範囲外

- 完了条件は、§4.1 だけで「任意 sibling」「パッケージからの直接結合禁止」
  「汎用 hooks レイヤによる自動検出」「欠落時も従来動作」の4点を判断できることとした。
- 「汎用 hooks レイヤ」は codd-gate を名指ししない共通の登録規約を持つ層と解釈した。
  登録規約、探索範囲、呼び出し順はこのタスクでは決めない。
- 対象文書は `docs/designs/codd-gate-design.md` にあり、変更許可された
  `tools/agent-project/` 配下ではない。書込制約を優先し、作業ツリーへは適用していない。
- 現 HEAD の codd-gate 専用 import、自動配線、関連テストは実装側の範囲外なので変更していない。

@followup 書込権限を持つ後続タスクで、成果候補を `docs/designs/codd-gate-design.md` §4.1 に適用する。
@followup 実装タスクで、汎用 hooks の登録規約と検出経路を定め、パッケージ内の codd-gate 専用結合を移す。

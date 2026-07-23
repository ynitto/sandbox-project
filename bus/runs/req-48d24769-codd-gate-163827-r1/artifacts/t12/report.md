切り口: 自動検出の手順と、検出後に値を渡す二つの汎用フックを分けて記述し、正典との対応を読み違えにくくした。

## 成果

`tools/agent-project/README.md` の「一貫性ゲート（codd-gate 連携・オプション）」を、設計書
`docs/designs/codd-gate-design.md` §4 と §4.1 に合わせて更新した。

- E2 `regression_cmd` を、毎タスクの verify PASS 後と done 確定前に動く「検証ゲート」の拡張と明記した。
- E3 `intake_cmd` を、負債から修復タスクを周期的かつ冪等に取り込む「backlog の自走」の pull 型拡張と明記した。
- 任意 sibling 部品の自動検出を「実体→バージョン→repos schema 互換性→対応機能」の短絡順とし、
  `codd_gate_routing.py` が実引数を組み立てる役割も記載した。

変更は README 1ファイル、5行追加・3行削除。agent_project と dashboard の実装、テスト、設計書には触れていない。

## 検証

- `git diff --check`: PASS。
- `git diff --name-only`: `tools/agent-project/README.md` のみ。
- `rg` で E2 `regression_cmd`、E3 `intake_cmd`、自動検出の短絡順、`codd_gate_routing.py` の記載を確認した。
- 文書だけの変更で実行時挙動は変わらないため、コードテスト、リンタ、型チェックは実行していない。

## 前提・未解決事項

- 完了条件は、README の対象節だけで、E2/E3 の正式名称と役割、自動検出レイヤの順序と実引数組み立てを
  §4・§4.1 と同じ意味にすること、と解釈した。
- 「自動検出」は agent_project の起動時配線ではなく、人か install 手順が生成ツールを明示起動した場合だけ
  sibling 部品内で行う検出を指す、とした。フックの有効化は引き続き YAML/CLI の設定で決まる。
- 未解決事項はない。既存実装に残る専用結合は t10 報告どおり範囲外のため変更していない。

@followup `agent_project` 内の専用 codd-gate 結合は、別タスクで §4.2 の決定的ゲートを満たすよう整理する。

切り口: 「検出できる」と「連携が有効になる」を分け、設定境界を一文で断定した。

## 成果

`tools/agent-project/README.md` の codd-gate 連携節に、連携を有効にする条件を明記した。

- YAML の `regression_cmd` / `intake_cmd`、または対応 CLI の `--regression-cmd` /
  `--intake-cmd` にコマンドを明示設定した場合だけ有効になる。
- sibling 部品による自動検出だけでは有効にならない。

変更は README 1 ファイル、3 行追加・2 行削除。agent_project、dashboard、テスト、設計書には触れていない。

## 検証

- `git diff --check`: PASS。
- `git diff --name-only`: `tools/agent-project/README.md` のみ。
- `rg` で YAML の両フィールド、対応する両 CLI、自動検出だけでは有効にならない旨が同じ段落にあることを確認した。
- 文書だけの変更で実行時挙動は変わらないため、コードテスト、リンタ、型チェックは実行していない。

## 前提・未解決事項

- 完了条件は、明示設定の対象を YAML と対応 CLI の両方について具体名で示し、自動検出との境界を
  README 単体で誤読できない形にすること、と解釈した。
- t12 の成果を前提に、E2/E3 の役割や自動検出手順は書き直さず、有効化条件の文だけを強めた。
- 未解決事項、範囲外で見つけた問題はない。

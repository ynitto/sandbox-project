# t10 統合結果

verify=pass

`tools/agent-dashboard/test/consistency-gate.test.js` のみを修正し、t9 の指摘を統合した。

- 片側だけ設定された実ペイロードで `regressionConfigured: true` / `intakeConfigured: false` を固定し、2 フラグの交差・取り違えを検出可能にした。
- 空白だけの `regression_cmd` / `intake_cmd` が configured / wired とも false、表示用 command が null になる境界契約を追加した。
- blocking 指摘と minor 指摘に矛盾・重複はなく、同一ファイルで一貫して検証できるため両方を採用した。
- 実装、UI、agent-project 本体、状態更新経路、公式契約外への書込みは変更していない。

検証:

- `node test/consistency-gate.test.js`: 7 passed
- `npm test`: PASS
- `git diff --check`: PASS
- 変更範囲: `tools/agent-dashboard/test/consistency-gate.test.js` の 1 ファイルのみ

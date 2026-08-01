# dashboard 一貫性ゲート修正報告

## 成果

- `tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md` の検証記録を最終実装に同期し、run 内部名、解消済みの CLI プレースホルダ不一致、完了済み `@followup`、誤った変更範囲の記述を除去した。
- `tools/agent-dashboard/test/consistency-gate-ui.test.js` で、agent-project README の sibling CLI から `--config` 引数を抽出し、画面表示と直接比較するようにした。README と UI のプレースホルダがずれれば失敗する。
- 製品ロジック、agent-project、状態書換経路は変更していない。変更は許可範囲内の2ファイルのみ。

## 検証

- `node test/consistency-gate-ui.test.js`: 成功（`consistency-gate-ui: ok`）
- `node test/consistency-gate.test.js`: 15 passed
- `node test/needs-gate-integration.test.js`: 10 passed
- `npx eslint test/consistency-gate-ui.test.js src/renderer/renderer.js`: 成功（No issues found）
- `git diff --check -- tools/agent-dashboard`: 成功
- `git status --short`: 上記2ファイルだけが変更済み

依存導入前は `yaml` 不在で隣接テストが開始できなかったため、`npm install --no-package-lock --ignore-scripts --no-audit --no-fund` で既存 `package.json` の依存だけを復元した。追跡対象ファイルは増えていない。

## 前提・未解決・範囲外

- 前提: judge の「修正して再検証」は、依存 UX 報告に残った2指摘を、最終状態との契約同期という単一原因として解消する要求と解釈した。完了条件は、両指摘の解消、README/UI 同期テストの成功、変更が `tools/agent-dashboard` 内に限定されることとした。
- 未解決: agent-project が実際に使用した明示 `--config` パスは現行 status 契約から取得できず、dashboard の自動探索候補との一致は断定できない。
- 範囲外: codd-gate の実在・互換性・実行成功の監視、agent-project のフック実装、UI からの設定・task status・done 書換は行っていない。

@followup 実効 `--config` を正確に表示する必要が生じた場合は、agent-project の status 契約拡張を別タスクで検討する。

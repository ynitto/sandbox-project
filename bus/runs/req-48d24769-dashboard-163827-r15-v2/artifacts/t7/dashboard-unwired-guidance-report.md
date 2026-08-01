# t7 成果報告: dashboard 未結線時ガイダンス

## 成果

- 既存の単一コンポーネント `consistencyGateHtml()` を維持し、`regression_cmd` または `intake_cmd` が未結線の場合だけ有効化導線を表示する契約を確認した。
- sibling CLI の案内を agent-project README の正典に合わせ、リポジトリルートから `python3 tools/agent-project/codd_gate_regression.py --config <状態 clone>/agent-project.yaml` を実行する表記へ修正した。
- dashboard README と UI テストも同じ表記へ同期した。設定編集、CLI 実行、タスク状態・done の書換経路は追加していない。
- 変更ファイルは `tools/agent-dashboard/README.md`、`src/renderer/renderer.js`、`test/consistency-gate-ui.test.js` の3件のみ。

## 検証

- `node test/consistency-gate-ui.test.js`: 成功。
- `node test/overview-ui.test.js`: 成功。
- `npm test`: 全件成功。
- `npx eslint src/renderer/renderer.js test/consistency-gate-ui.test.js test/overview-ui.test.js`: 成功。
- `git diff --check`: 成功。
- `npm run lint`: 既存の `src/features/cowork/main/cowork.js:590` の未使用変数 `cfg` で失敗。本変更対象の lint は成功しており、この無関係な既存問題は修正していない。

## 前提・未解決・範囲外

- 「未設定時」は依存設計書に従い、値が空の場合だけでなく、別コマンドが設定され codd-gate の正規形に合わない「未結線」も含むと解釈した。
- sibling CLI は未設定の YAML `regression_cmd` と既存設定ファイルがある場合だけ案内する。JSON、設定ファイル未検出、既存 `regression_cmd` ありでは案内せず、設定編集のみを示す。
- 設定ファイルを開く既存操作は `openPath` のみで、保存やコマンド実行は利用者に委ねる。
- agent-project のフック実装、新規 CLI、設定・needs・status・done の書換は範囲外として未実施。
- 未解決は package 全体 lint の上記既存エラーのみ。

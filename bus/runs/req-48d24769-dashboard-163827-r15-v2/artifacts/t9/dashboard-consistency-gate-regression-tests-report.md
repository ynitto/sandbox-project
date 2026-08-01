# t9 成果報告: dashboard 一貫性ゲート回帰テスト

## 成果／サマリー

`tools/agent-dashboard/test/needs-gate-integration.test.js` だけを変更し、2件の回帰テストを追加した。

- `readProject().consistencyGate` 相当の公式表示モデルについて、両方設定済み、片方未設定、両方未設定の3状態と表示ラベルを固定した。
- renderer が公式 `consistencyGate` payload 外の生設定を読まず、描画時に payload を変更しないことを固定した。
- 有効化導線は `openPath` だけを呼び、未設定状態を楽観的に書き換えないことを固定した。
- 公式 `needs/<id>.md` の `failure-*` frontmatter を `parseNeeds()` で読み、codd-gate の regression 失敗要約と一貫性ゲート案内を表示しつつ、`kind=blocked`、`decided=false` を保つことを固定した。

本番コード、テスト基盤、スナップショット、agent-project 本体は変更していない。

## 検証内容と結果

- `node test/needs-gate-integration.test.js`: 10件成功（追加2件を含む）。
- `node test/consistency-gate.test.js`: 15件成功。
- `node test/consistency-gate-ui.test.js`: 成功。
- `node test/needs-diagnosis.test.js`: 12件成功。
- `npm test`: 全テスト成功。
- `npx eslint test/needs-gate-integration.test.js`: 成功。
- `git diff --check`: 成功。
- `npm run lint`: 既存の `src/features/cowork/main/cowork.js:590` の未使用変数 `cfg` 1件だけで失敗。本タスク外のため変更していない。
- worktree の差分は `test/needs-gate-integration.test.js` 1ファイルのみ。

## 採用した前提・未解決事項・範囲外

- 「関連する単一テストモジュール」は、概要の結線状態と needs の失敗表示を同時に扱う既存の `needs-gate-integration.test.js` と解釈した。
- 「設定済み／片方未設定／両方未設定」は、公式表示モデルの `*Configured`、`*Wired`、`wired` が表す、両フック設定済み、片方のみ設定済み、両方未設定の3状態と解釈した。
- 「公式契約だけを読む」は、概要 UI は `readProject().consistencyGate`、要対応 UI は `parseNeeds()` が返す `failure-*` を入力にし、生 YAML や独自状態を renderer から読まないことと解釈した。
- 「UIから状態を書き換えない」は、描画で payload を変更せず、有効化操作は `openPath` のみで、`blocked` / `decided=false` を維持することと解釈した。
- 未解決は package 全体 lint の既存エラーのみ。
- @followup: `cowork.js:590` の未使用変数 `cfg` は別タスクで利用意図を確認して除去する。

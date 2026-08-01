# dashboard 一貫性ゲート最終検証

verify=fail

## 独立検証結果

- 最終 HEAD: `f2042ebfe458674b75b5afcde124e98d8ce40f3a`
- 作業ツリー: clean（依存復元後も追跡・未追跡差分なし）
- `main...HEAD`: `tools/agent-dashboard` 内の5ファイルのみ、226 insertions / 11 deletions。スコープ外変更なし
- 関連テスト: `consistency-gate.test.js` 15件、`needs-gate-integration.test.js` 10件、`needs-diagnosis.test.js` 12件が成功。`consistency-gate-ui.test.js` も成功
- 全テスト: `npm test` 成功（exit 0）
- 対象 ESLint: 成功（exit 0）
- 全 lint: `npm run lint` 失敗（exit 1）。`src/features/cowork/main/cowork.js:590` の未使用変数 `cfg` 1件
- 差分品質: `git diff --check main...HEAD -- tools/agent-dashboard` 成功。変更 Markdown に末尾空白なし、競合マーカーなし
- terminal 出力: 再実行した検証プロセスに `ok:false` なし。初回は依存未導入で停止したため、既存依存を `npm install --no-package-lock --ignore-scripts --no-audit --no-fund` で復元後、新規プロセスで再実行した
- UX: README と画面の sibling CLI および `--config <状態 clone>/agent-project.yaml` の直接照合テストは成功。概要の3状態、未結線時の設定導線、needs の失敗要約も関連テストで確認
- done 不変条件: 有効化操作は `api.openPath()` のみ。`main...HEAD` に設定・needs・status・done の書換経路はなく、公式 needs から読んだ `kind=blocked` / `decided=false` の保持テストも成功

## issues

- `tools/agent-dashboard/src/features/cowork/main/cowork.js:590` の `const cfg = config.cowork || {};` が未使用で、全体 ESLint が失敗する。宣言を削除し、`npm run lint` と `npm test` を再実行すること。この行は今回差分外だが、競合解消後の最終状態で「全検証成功」を満たしていない。
- (minor) `tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md:19` に run 内部名 `t3` / `t4` が残っている。UX 指摘の意図どおり永続設計書から内部ノード名を除き、「調査結果および現物コード」等へ置き換える。

{"ok": false, "issues": ["tools/agent-dashboard/src/features/cowork/main/cowork.js:590 の未使用変数 cfg により npm run lint が exit 1。宣言を削除して全 lint と全テストを再実行すること。", "(minor) tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md:19 に run 内部名 t3 / t4 が残っているため、永続設計書から除去すること。"]}

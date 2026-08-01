# dashboard 一貫性ゲート最終検証

verify=fail

## 独立検証結果

- 最終 HEAD: `f2042ebfe458674b75b5afcde124e98d8ce40f3a`
- 作業ツリー: clean
- `main...HEAD` の変更: `tools/agent-dashboard` 内の5ファイルのみ（226 insertions / 11 deletions）
- 関連検証: `consistency-gate.test.js` 15件成功、`consistency-gate-ui.test.js` 成功、`needs-gate-integration.test.js` 10件成功
- 全テスト: `npm test` 成功（exit 0）
- 全 lint: `npm run lint` 失敗（exit 1）
- 差分品質: `git diff --check main...HEAD -- tools/agent-dashboard` 成功、競合マーカーなし、作業ツリー内に `terminal ... ok:false` なし
- UX: README と画面の sibling CLI / `--config` 引数が一致し、概要の3状態、未結線時の設定導線、needs の失敗要約を確認
- done 不変条件: 設定導線は `shell:openPath` のみ。変更差分に設定・needs・status・done の書換経路なし。blocked/undecided を保つ統合検証も成功

## issue

- `tools/agent-dashboard/src/features/cowork/main/cowork.js:590` の `cfg` が未使用で、全体 ESLint が `no-unused-vars` エラーになる。このファイルは今回の `main...HEAD` 差分外だが、競合解消後の最終状態で「全検証成功」を満たさない。許可範囲内の後続修正で未使用宣言を削除し、`npm run lint` を再実行すること。

{"ok": false, "issues": ["tools/agent-dashboard/src/features/cowork/main/cowork.js:590 の未使用変数 cfg により npm run lint が失敗する。未使用宣言を削除して全 lint を再実行すること（今回差分外）。"]}

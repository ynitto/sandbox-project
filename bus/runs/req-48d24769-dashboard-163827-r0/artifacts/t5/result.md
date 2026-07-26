# t5 成果報告

## 成果／サマリー

- `tools/agent-dashboard/src/renderer/renderer.js` の `consistencyGateHtml()` だけを表示側の結合点として変更した。
- t4 が追加した `regressionConfigured` / `intakeConfigured` を使い、各フックを「設定: あり／なし」と明示するようにした。古い main からフラグが来ない場合だけ既存コマンド値へフォールバックする。
- codd-gate 以外のコマンドが設定済みなら、値を表示したまま「一貫性ゲートの検査ではない」と示す。
- 結線済みの回帰検査にも「失敗は、done 前に解消すべきドキュメントとコードの不整合が残っていることを意味する」と表示する。
- 既存の設定ファイルを開く読取専用導線、`codd_gate_regression.py` sibling CLI、`intake_cmd` の yaml 直接編集案内は維持した。UI から状態・設定を書き換える経路は追加していない。
- `test/consistency-gate-ui.test.js` に設定有無、別コマンド、未設定、失敗説明の表示確認を追加した。

## 検証内容と結果

- `node test/consistency-gate-ui.test.js`: PASS
- `node test/consistency-gate.test.js`: PASS（6 tests）
- `node test/needs-gate-integration.test.js`: PASS（8 tests）
- `node test/needs-diagnosis.test.js`: PASS（13 tests）
- `npm test`: PASS（終了コード 0、agent-dashboard 全テスト）
- `git diff --check`: PASS
- `git status --short`: 変更は `tools/agent-dashboard/src/renderer/renderer.js` と対応テストだけ。許可範囲外の変更なし。

## 採用した前提・未解決事項・範囲外

- 完了条件は、概要の一貫性ゲート表示で「設定有無」「codd-gate 結線状態」「未結線時の設定編集／sibling CLI」「回帰失敗の意味」を同時に判断でき、既存 needs 要約を維持すること、と解釈した。
- 「設定あり」は t4 の明示フラグを正とした。後方互換時だけ trim 後のコマンド値の有無で補う。
- 既存の needs 表示が codd-gate と通常の検証失敗を区別し、要約・対処・context を保持しているため、needs 側は変更しなかった。
- 未解決事項なし。agent-project 本体、done 判定、状態更新 API、独自契約は変更していない。

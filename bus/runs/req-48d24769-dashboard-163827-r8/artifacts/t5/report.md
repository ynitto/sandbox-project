# t5 成果報告

## 成果／サマリー

- 完了条件を「`tools/agent-dashboard/src/renderer/renderer.js` の既存概要 UI 結合点で、`regression_cmd` / `intake_cmd` の設定有無と、一貫性ゲートへの結線状態・役割を読み取り専用で判別できること」と定めた。
- 指定 worktree には完了条件を満たす実装がすでに存在したため、重複するソース変更は行っていない。
- `consistencyGateHtml()` は各フックについて「設定: あり／なし」と「結線済み／未結線」を別々に表示する。別の汎用コマンドが設定された場合も、一貫性ゲートではないことを明示する。
- `regression_cmd` は失敗時に `done` の確定を止める完了前回帰検査、`intake_cmd` は検出したズレを修復タスクとして取り込む経路として説明される。
- 未結線時は既存の概要セクション様式で、README と同じ設定行、設定ファイルを開く読み取り専用導線、利用可能な sibling CLI の案内を表示する。UI から設定・needs・状態を変更しない。
- 概要への挿入とイベント結線は既存 `renderOverview()` の `${consistencyGateHtml(p)}` と `bindConsistencyGate(el)` を再利用している。

## 検証内容と結果

- `cd tools/agent-dashboard && npm test`: 成功（失敗 0）。`consistency-gate.test.js`、`consistency-gate-ui.test.js`、`needs-gate-integration.test.js` を含む全テストが通過した。
- `node test/consistency-gate-ui.test.js`: 成功（`consistency-gate-ui: ok`）。設定有無、全結線／一部結線／未結線、別コマンド、README と同じ有効化導線、HTML エスケープを確認した。
- `git diff --check`: 成功。
- `git status --short`: 出力なし。指定 worktree に未コミット変更はない。

## 採用した前提・未解決事項・範囲外

- 前提: 依存成果 t2/t3 と現物コードの一致を確認し、既存実装を削除・再実装するより、そのまま採用することが最小かつ安全と判断した。
- 前提: 「設定有無」と「一貫性ゲートへの結線」は別概念である。`regression_cmd` / `intake_cmd` は汎用フックのため、非空だけでは結線済みと断定しない。
- 未解決事項: なし。
- 範囲外: `project.js`、needs 診断表示、状態変更操作、新規 UI 基盤、agent-project 本体のフック実装には変更を加えていない。

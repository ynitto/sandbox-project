# overview 一貫性ゲート描画の調査結果

## 完了条件と結論

- 完了条件は、概要画面で既存のラベル・状態表示・案内リンクのパターンを確認し、`regression_cmd` / `intake_cmd` の設定・結線状態と有効化導線を置く最小箇所を特定すること、と解釈した。
- target main (`3756b4f07cf3d0c9c6225cd1f7f6092ff27dc9f2`) には対象表示がすでに実装・結線済みだったため、重複変更は行っていない。

## 特定した最小描画箇所

- 表示モデルの HTML 化: `tools/agent-dashboard/src/renderer/renderer.js:1190` の `consistencyGateHtml(p)`。
- 概要への挿入点: `tools/agent-dashboard/src/renderer/sections/overview.js:341`。概要の「現在の状態 / あなたの対応 / 進捗 / 成果」の直後、計画バージョンの直前で `${consistencyGateHtml(p)}` を 1 回だけ描画する。
- 読み取り専用の案内ボタン結線: `tools/agent-dashboard/src/renderer/renderer.js:1307` の `bindConsistencyGate(root)` を `tools/agent-dashboard/src/renderer/sections/overview.js:358` から呼ぶ。`api.openPath()` で自動検出した設定ファイルを開くだけで、設定値・タスク状態・done は変更しない。

## 既存パターンとの整合

- 状態は既存の `badge info` / `badge warn` を再利用し、全体を「結線済み / 一部結線 / 未結線」、各フックを「結線済み / 未結線」、設定値を「設定: あり / なし」で区別する。
- 案内は既存の `need-resolution`、`label-chip`、`summary-actions`、`summary-link secondary` を再利用する。
- `regression_cmd` は「失敗すると done の確定を止める」、`intake_cmd` は「検出したズレを修復タスクとして積む」と説明する。別コマンドが設定済みなら未設定とは扱わず、「一貫性ゲートの検査ではない」と明記する。
- 未結線時のみ README と同じ設定例を表示する。既存 YAML の `regression_cmd` が空なら sibling CLI を案内し、`intake_cmd` は直接編集を案内する。設定ファイルが判明している場合だけ「自動検出した設定ファイルを開く」を表示する。

## 検証

- `node test/consistency-gate-ui.test.js`: 成功。
- `node test/overview-ui.test.js`: 成功。
- `node --check src/renderer/renderer.js`: 成功。
- `node --check src/renderer/sections/overview.js`: 成功。
- `npm test`: 失敗。変更起因ではなく、依存パッケージ `yaml` が未導入のため `test/feature-split.test.js` のロード時に `MODULE_NOT_FOUND`。その前の `ui-copy` と feature-split の初期チェックは成功。
- `git diff -- tools/agent-dashboard`: 差分なし。

## 前提・未解決・範囲外

- `p.consistencyGate` は main 側で構築済みの読み取り専用表示モデルであり、renderer は真偽判定や設定探索を再実装しない前提。
- dashboard が表示するのは自動探索した設定候補であり、agent-project が `--config` で別設定を使う場合の一致は断定できない。この制約は既存 UI に表示済み。
- `main/project.js`、needs-diagnosis、設定書換操作は調査・変更対象外とした。
- 範囲外で追加修正が必要な問題は見つからなかった。全テスト実行には依存導入が必要だが、設定・依存を書き換えない制約に従い実施していない。

# t5 renderer 一貫性ゲート表示

## 成果／サマリー

完了条件を「概要で `regression_cmd` / `intake_cmd` の設定有無と一貫性ゲートの結線状態を判別でき、未結線時は設定編集または利用可能な sibling CLI を案内し、既存の codd-gate／回帰失敗要約を維持する」とした。

指定 worktree の `tools/agent-dashboard/src/renderer/renderer.js` には、すでに次が実装済みだったため追加変更は行わなかった。

- `consistencyGateHtml()` が両設定の「設定あり／なし」「結線済み／未結線」、現在のコマンド、および全体の「結線済み／一部結線／未結線」を表示する。
- 未結線時は README と同じ YAML 行を示し、既存設定ファイルは `bindConsistencyGate()` から既定エディタで開ける。
- `regression_cmd` は既存設定ファイルがある場合だけ sibling CLI `codd_gate_regression.py` を案内し、`intake_cmd` は対応 CLI がないため直接編集を案内する。
- UI は設定を保存・書換せず、main が返した `consistencyGate` 表示モデルを描画するだけである。
- needs 側の一貫性ゲート節と既存の「検証失敗」要約・context・「判断材料を見る」折り畳みが共存する。

Ponytail の「既存実装を再利用し、不要なコードを書かない」に従い、すでに完了条件を満たす実装へ重複差分を追加しなかった。作業ツリーは clean のまま。

## 検証内容と結果

- `tools/agent-dashboard` で `npm test`: exit 0、全件成功。
- `node test/consistency-gate-ui.test.js`: 成功。
- `node test/needs-gate-integration.test.js`: 8 passed。
- `node test/overview-ui.test.js`: 成功。
- `node test/needs-diagnosis.test.js`: 13 passed。
- `git diff --check`: 成功。
- `git status --short`: 出力なし（clean）。

完了条件との照合:

1. 設定状態の可視化: `regression_cmd` / `intake_cmd` 各行で設定有無と値を表示済み。
2. 一貫性ゲート状態の可視化: 各フックと全体の結線状態を表示済み。
3. 未結線時の対処導線: 設定編集、regression 用 sibling CLI、intake の直接編集案内を表示済み。
4. 既存可読性の維持: overview・needs 統合・needs 診断テストで確認済み。
5. スコープ遵守: リポジトリ内ファイルの変更なし。バックエンド契約・状態書換・独自設定保存には触れていない。

## 採用した前提・未解決事項・範囲外

- 前提: 現ブランチに既に存在する実装も本タスクの成果として評価し、不足がない場合は重複実装しない。
- 前提: `consistencyGate` の結線判定は main 側の公式表示モデルを正とし、renderer では再判定しない。
- 未解決事項: なし。
- 範囲外で見つけた問題: なし。


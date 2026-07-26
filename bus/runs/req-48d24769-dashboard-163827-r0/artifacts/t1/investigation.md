# 調査結果: agent-dashboard 一貫性ゲート表示経路

## 完了条件と採用した前提

- 完了条件は、概要・プロジェクト情報・needs-diagnosis の実データ経路を追い、`regression_cmd` / `intake_cmd` の結線状態と、未結線時の読み取り専用の有効化導線を追加するための最小変更点を特定すること。
- この work は「調査し、最小の変更点を特定する」と解釈し、既存実装が条件を満たす場合は重複実装しない。
- 「結線済み」はキーが非空という意味ではなく、`regression_cmd` が `codd-gate verify ... --base`、`intake_cmd` が `codd-gate tasks ... --debt` を指すこととした。両キーは汎用フックであり、例えば `make -s smoke` は一貫性ゲート結線ではない。
- 有効化導線は設定ファイルを開くことと README 同等の設定行／sibling CLI を表示することまで。UI から設定・needs・状態を書き換えない。

## 結論

指定ブランチには必要な最小経路がすでに実装されているため、今回の worktree にはソース変更を加えなかった。追加すべき箇所は以下の 4 点で、現状はすべて満たしている。

1. 読み取りモデル  
   `src/features/agent-project/main/project.js:1609` の `consistencyGateStatus()` がワークスペース由来の agent-project 設定だけを読み、`regressionWired` / `intakeWired` / `wired`、表示用コマンド、設定ファイルの実パスを返す。`readProject()` が `:1763` で `project.consistencyGate` として renderer へ載せる。グローバル設定を別プロジェクトの状態として誤採用しない。

2. 概要／プロジェクト情報の表示  
   `src/renderer/renderer.js:1090` の `consistencyGateHtml()` が 2 フックを「結線済み／未結線」、全体を「有効／一部のみ／未結線」で表示する。未結線時は README と同じ設定行を示し、既存設定ファイルがあれば開くボタンを出す。`regression_cmd` には `codd_gate_regression.py --config ...` と `--dry-run` の導線を示し、対応 CLI のない `intake_cmd` は yaml 直接編集と明示する。`bindConsistencyGate()` (`:1185`) は `api.openPath` のみで、書込みを行わない。  
   実際の配置は分割後の `src/renderer/sections/overview.js:349`（描画）と `:366`（binding）。技術情報へ重複表示せず、通常利用者が最初に見る概要を正本としている。

3. needs-diagnosis との接続  
   `src/features/agent-project/main/project.js` の `parseNeeds()` が producer の構造化 frontmatter を `failureSummary` / `failureResolution` / `failureContext` として運び、旧票だけ散文診断へフォールバックする。  
   `src/renderer/sections/needs.js:1211` の `needFailureViewModel()` は解析済み事実だけを表示モデル化する。`:1307` の `needGateSource()` は、結線済み `regression_cmd` による回帰失敗と、タスク自身の codd-gate verify を区別する。`:1316` の `renderNeedFacts()` は既存の失敗要約・対処・context を保ったまま、「一貫性ゲート」の意味、`intake_cmd` の結線状態、必要時の設定ファイルを開く導線を追記する。`:1608-1609` で gate/diagnosis を描画 signature に含め、更新が再描画される。

4. 回帰防止  
   `test/consistency-gate.test.js` が main 側の結線判定、`test/consistency-gate-ui.test.js` が概要表示と README 同等導線、`test/needs-diagnosis.test.js` が診断情報との共存、`test/needs-gate-integration.test.js` が needs 詳細への統合、`test/overview-ui.test.js` が概要への配置を固定している。

## 影響範囲

- データ producer: `src/features/agent-project/main/project.js`
- 共通表示 helper: `src/renderer/renderer.js`
- 概要の挿入点: `src/renderer/sections/overview.js`
- needs の挿入点: `src/renderer/sections/needs.js`
- IPC は既存の `openPath` を再利用し、新規書込み API は不要。
- `src/main/project.js` は feature 実体への互換 shim なので、同じ実装を複製する変更は不要。

## 検証

- 対象テスト:
  - `node test/consistency-gate.test.js` — 6 passed
  - `node test/consistency-gate-ui.test.js` — pass
  - `node test/needs-diagnosis.test.js` — 13 passed
  - `node test/needs-gate-integration.test.js` — 8 passed
  - `node test/overview-ui.test.js` — pass
- 全体テスト: `npm test` — exit 0、全テスト成功。
- `git status --short` — 空。指定 worktree に未コミット変更なし。

## 未解決事項・範囲外

- 未解決事項なし。
- agent-project 本体のフック実装、UI からの状態・設定書換、公式 needs/inbox/commands 契約外への書込みは調査・変更していない。
- 現状の実装は先行コミットに含まれている。今回さらに変更すると同一機能の重複になるため、Ponytail/YAGNI に従い変更なしとした。

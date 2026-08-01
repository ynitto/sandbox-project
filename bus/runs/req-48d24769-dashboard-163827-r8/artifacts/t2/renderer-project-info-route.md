# renderer.js 概要・プロジェクト情報表示経路 調査結果

## 成果／サマリー

調査基準は、依存タスク t1 が示したローカル参照 `origin/main`（`652184e15`、2026-08-01 10:34:29 +0900 時点）とした。指定 worktree の HEAD `d6e2c6d20` は `origin/main` と大きく乖離し、一貫性ゲート表示の先行実装を含むため、現行 main の経路確認には使わず、結合点候補の裏付けにだけ用いた。

現行 main のプロジェクト詳細取得・描画は次の一本の経路である。

1. `src/renderer/renderer.js:855-864` の `selectProject(dir)` が選択先を `state.selectedDir` に保存し、`reloadProject()` を呼ぶ。
2. `src/renderer/renderer.js:866-900` の `reloadProject()` が `api.readProject(state.selectedDir)` を呼ぶ。
3. `src/features/agent-project/preload.js:12` がこれを `dashboard:project` IPC に変換する。
4. `src/features/agent-project/main/ipc.js:56-60` が `project.readProject(dir, loadConfig())` を呼ぶ。
5. `src/features/agent-project/main/project.js:1799-1957` の `readProject()` が backlog、needs、設定関連情報、bus 情報等を整形した単一スナップショットを返す。
6. `reloadProject()` がその戻り値を `state.project` に格納し、`renderHeader()` と `renderAllTabs()` を呼ぶ。
7. `renderAllTabs()`（`renderer.js:1070-1087`）から概要 `renderOverview()` とプロジェクト設定 `renderProjectSettings()` が再描画される。

概要の実体は現行 main では `renderer.js` 内ではない。`src/renderer/index.html:669-688` の順で core の `renderer.js`、`sections/overview.js`、最後に bootstrap がクラシックスクリプトとして読み込まれ、同一グローバルスコープを共有する。概要 HTML は `src/renderer/sections/overview.js:265-353` の `renderOverview()` が `state.project` を読み、`#tab-overview` へ一括描画する。

プロジェクト情報（診断情報）の経路は、`sections/overview.js:355-407` の `renderProjectSettings()` が「診断情報を開く」ボタンを結線し、`renderer.js:1163-1225` の `technicalProjectInfoHtml()` / `openTechnicalInfo()` が `state.project` を読み、`#technical-project-info` ダイアログへ描画する。

## 最小の既存結合点

推奨は概要への追加である。人が通常見る画面で未設定に気づけ、既存の `state.project` と再描画周期をそのまま使える。

- 取得・整形: `project.js` の `readProject()`。同ファイルは既に `readToolConfig` を import 済みで、`resolveBusDir()` 内の `project.js:1748-1752` では `readToolConfig('agent-project', [workspace, ...agentDirCandidates(workspace)])` により実効設定を読んでいる。`toolconfig.js:24-59` は YAML/YML/JSON のトップレベルスカラを読み、設定ファイルパスも返す。したがって新規パーサ、新規 IPC、新規 preload API は不要で、ここで `regression_cmd` / `intake_cmd` の非空有無（必要なら値と設定ファイルパス）を既存スナップショットへ載せるのが最小。
- 描画: `sections/overview.js:286-337` の `renderOverview()` テンプレート。既存の概要セクション列に小さな読み取り専用セクションを差し込める。イベントが必要なら同関数末尾 `339-352` が既存のボタン結線箇所。
- 設定ファイルを開く導線: 既存 `api.openPath(...)` を使う。新規の書換 API は不要。UI は状態を書き換えず、README の設定編集または sibling CLI を案内するだけに留められる。

代替のさらに狭い表示先は `renderer.js:1191-1197` の `technicalProjectInfoHtml()` 内 `<dl class="developer-facts">`。2 行の事実表示だけならここが最小差分だが、設定画面の「調査と高度な設定」配下であり、未結線を日常的に発見して有効化を支援する目的には概要より弱い。

設定有無だけを示す場合の最小データ形は、たとえば `regressionConfigured` / `intakeConfigured` の boolean 2 個で足りる。有効化導線まで出すなら `configFile` も必要。コマンド本文は診断目的がなければ返さない方が表示・漏えい面で小さい。なお「一貫性ゲートが結線済み」を表示する場合は単なる設定有無と意味が異なる。`regression_cmd` / `intake_cmd` は汎用フックなので、非空だけで codd-gate 有効と断定してはいけない。

README の現行 main の正典は `tools/agent-project/README.md:347-373`。未結線時の案内は次と一致させる。

- `regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'`
- `intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'`
- sibling CLI `python3 tools/agent-project/codd_gate_regression.py --config .agent/agent-project.yaml` は `regression_cmd` のみを冪等注入する。`intake_cmd` は設定編集が必要。

指定 worktree の先行実装も、同じ結合点（`readProject()` の追加フィールド、`renderer.js` の表示 helper、`sections/overview.js` の 1 箇所差し込み）を使っている。ただしそのまま移植対象とはせず、現行 main 上で再適用・再検証すべきである。

## 検証内容と結果

- `.codegraph/` が無いことを確認し、`origin/main` のソースを `git show` / `git grep` で直接再導出した。
- preload → IPC → `readProject()` → `state.project` → `renderAllTabs()` → 概要／診断情報の各呼出経路を、上記ファイルと行番号で照合した。
- `origin/main` には `regression_cmd` / `intake_cmd` の dashboard 表示実装が存在しないことを `git grep` で確認した。
- `readToolConfig()` が YAML/YML/JSON のトップレベルスカラと設定ファイルパスを返すことを確認した。
- README の有効化コマンドと sibling CLI の責務を確認した。
- 調査タスクのためテストは実行していない。コード変更は無く、指定リポジトリの `tools/agent-dashboard` は調査前後とも変更なし。

## 前提・未解決事項・範囲外

- 「現行 main」はネットワーク fetch をせず、依存成果に明記されたローカル `origin/main` と解釈した。git 規約に従い checkout / rebase / commit / push は行っていない。
- 完了条件は、表示経路と最小結合点の特定および README と同じ有効化導線の確認であり、実装は含めないと解釈した。
- `state.project` へフィールドを追加することは既存 IPC の後方互換な拡張で済むが、厳密には dashboard 内部の返却形変更である。「バックエンド契約の変更」を一切許さない解釈なら renderer だけではプロジェクト設定を取得できず、正確な表示は不可能。この場合は別タスクで読み取り専用フィールド追加の許可が必要。
- needs 診断ロジック、agent-project 本体のフック実装、UI からの設定・done 状態の書換は調査も変更もしていない。

@followup 実装タスクでは、現行 main 上で `readProject()` の読み取り専用設定状態、概要セクション、表示ロジックの最小テストだけを追加し、先行ブランチの大規模差分は丸ごと移植しないこと。

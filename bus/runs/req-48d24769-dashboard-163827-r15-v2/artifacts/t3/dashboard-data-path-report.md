# tools/agent-dashboard 状態表示・データ経路調査

## 完了条件と採用した前提

- 完了条件は、`regression_cmd` / `intake_cmd` の設定、codd-gate 由来の失敗、`needs/`・`inbox/`・`commands/` の公式契約が画面へ届く経路を現物で特定し、状態表示を置く最小の既存コンポーネントと、変更時に維持すべき可読性を後続作業が判断できる形で示すこと、とした。
- t2 の統合報告どおり、現 HEAD は最新 main と同内容である前提で調査した。現 HEAD には目的の表示がすでに実装・文書化されているため、重複する UI・データモデルは追加しないのが最小変更と判断した。
- 「公式契約外データへアクセスしない」は、設定ファイル、`needs/*.md`、`inbox/`、`commands/*.err`、`commands/processed/*.json` と既存 IPC スナップショットだけを根拠にすること、と解釈した。codd-gate の実行、バイナリ探索、独自ログ解析、agent-project の状態書換は対象外とした。

## 成果・データ経路

### 1. regression_cmd / intake_cmd と概要表示

経路は次の一本である。

`agent-project.{yaml,yml,json}` → `readToolConfig()` → `consistencyGateStatus()` → `readProject().consistencyGate` → `dashboard:project` IPC → preload の `api.readProject()` → `reloadProject()` の `state.project` → `consistencyGateHtml()` → `renderOverview()`

- 設定探索: `src/features/agent-project/main/toolconfig.js:38`。ワークスペース候補を優先し、最後に `~/.agents` を見る。最初の設定が壊れていても別スコープへ倒さない。
- 判定モデル: `src/features/agent-project/main/project.js:1780-1820`。汎用フックの「設定あり」と codd-gate への「結線済み」を分離し、`regression_cmd` は `codd-gate verify ... --base`、`intake_cmd` は `codd-gate tasks ... --debt` の語順だけを判定する。コマンドは実行しない。
- スナップショット: `src/features/agent-project/main/project.js:1940,2001` で設定を読み、`consistencyGate` として返す。
- IPC: `src/features/agent-project/main/ipc.js:56-60` → `src/features/agent-project/preload.js:12` → `src/renderer/renderer.js:869-873`。
- 表示本体: `src/renderer/renderer.js:1188-1311`。両フックの「設定あり/なし」「結線済み/未結線」「現在値」と、全体の「結線済み/一部結線/未結線」を表示する。未結線時だけ README と同じ YAML/JSON 設定例、`regression_cmd` 用 sibling CLI、`intake_cmd` は直接編集する旨を示す。設定ファイルを開く操作だけで、書換はしない。
- 配置: `src/renderer/sections/overview.js:265-360` の概要末尾で `consistencyGateHtml(p)` を差し込む。ここが「現在の状態と次の一手」を扱う既存ハブなので、状態表示を置く最小の既存コンポーネントである。`technicalProjectInfoHtml()` は開発者向け細目、needs は個別失敗の文脈なので、全体状態の主表示にはしない。

制約も画面へ明記済みである。dashboard の自動探索候補しか分からず、agent-project が `--config` で別設定を使うか、codd-gate の実在・互換性・実行成功までは公式契約から確認できない。

### 2. needs / codd-gate 失敗 / needs-diagnosis

経路は次の一本である。

`needs/<id>.md` の `failure-*` frontmatter → `parseNeeds()` → `readProject().needs[]` → `state.project.needs[]` → `needFailureViewModel()` / `needGateSource()` → `renderNeedFacts()` → `renderNeedDetail()`

- 契約読込: `src/features/agent-project/main/project.js:716-822`。producer が書いた `failure-summary`、`failure-resolution`、`failure-category/owner/command/workdir/exit/target`、`failure-phase` を表示モデルへ移す。構造化フィールドが無い旧票だけ `_diagnoseFailure()` (`:596`) で後方互換解析する。
- 表示モデル: `src/renderer/sections/needs.js:1350-1380`。解析済みの `failureSummary` / `failureContext` だけを断定表示に使い、不明なら要約を捏造せず生の判断材料を残す。`verify 未定義` は失敗扱いしない。
- codd-gate 帰属: `needGateSource()` (`:1458`) は、記録された codd-gate コマンドと `failurePhase=regression` またはタスク verify の組合せでだけ由来を判定する。現在の結線状態だけから過去の失敗原因を推測しない。
- 個別表示: `renderNeedFacts()` (`:1488`) は既存の順序を保ち、`検証失敗` 要約 → `確認・対処` → context（分類、対処対象、コマンド、作業場所、終了コード、確認対象）を先に出し、その後ろにだけ `一貫性ゲート` ブロックを足す。regression 経路なら done 確定を止めた意味、verify 経路なら regression とは別経路であること、さらに `intake_cmd` が未結線なら概要の有効化導線へ案内する。
- 詳細カード: `renderNeedDetail()` (`:1796-1855`) の既存 `状況` セクションが最小配置先である。失敗の意味と対処を、操作欄や折り畳みの生証拠から分離して最初に読める。
- needs-diagnosis: `canDiagnoseNeed()` / `needAssistActionsHtml()` (`:1375-1410`) は、構造化失敗または限定的な失敗らしい散文がある場合に `AIと対話で診断` と `文面を生成` を出す。断定表示の判定とは分離されており、診断ボタンを出す推測が事実表示を汚染しない。実行側は `src/features/agent-project/main/agent.js:533-548,648-710` の読み取り専用 `failure-diagnosis` モードで、修正・再実行は提案だけに制限される。

### 3. commands / inbox 契約からの表示

- `commands/*.err` → `listCommandFailures()` (`project.js:855-881`) → task id ごとの最新 `commandFailure` → `commandFailureHtml()` (`needs.js:166`)。失敗理由と対処指示を「過去の操作」として表示し、カードを送信済みに隠さず再操作可能にする。
- `commands/processed/*.json` → `listCommandReceipts()` (`project.js:883-912`) → 最新 `commandReceipt` → `commandReceiptHtml()` (`needs.js:186`)。受理済みを一時表示し、失敗があれば失敗表示を優先する。
- `commands/*.json` の未処理送信は、needs ファイルの path+mtime と localStorage の 15 分 TTL (`needs.js:16-44`) で「送信済み（取り込み待ち）」を示す。これは公式ファイル内容を補う UI フィードバックで、タスク状態を書き換えない。
- `inbox/*.{json,md,markdown,txt}` → `readProject().inboxFiles` (`project.js:1919-1922,1972`) → `src/renderer/sections/backlog.js:545` の「追加待ち N」バッジ。inbox はバックログ化前なので needs や概要のゲート状態へ混ぜない。
- dashboard からの操作は既存 `src/features/agent-project/main/actions.js` が `commands/` または `inbox/` へ投函する。表示側は次回 `reloadProject()` で公式契約を再読込するだけであり、done 不変条件を迂回する直書き経路はない。

### 維持すべき可読性

1. 全体状態は概要、個別失敗は needs、取り込み前タスクはバックログに置き、同じ情報を複数箇所で主表示しない。
2. 「設定あり」と「codd-gate 結線済み」を混同しない。別コマンドが設定済みなら値を隠さず、「一貫性ゲートの検査ではない」と示す。
3. 状態・意味・次の一手の順で読む。needs では既存の失敗要約と context を先に保ち、ゲート説明は後置する。
4. 断定は構造化 `failure-*` と記録コマンドに限定する。不明は不明のまま生証拠へ導き、AI 診断を事実源にしない。
5. 成功受理、取り込み待ち、取り込み失敗を区別し、失敗を成功表示や localStorage の送信済み表示で覆わない。
6. 有効化は README と同文脈の設定編集/sibling CLI 案内まで。UI から設定・needs・done を変更しない。

## 検証内容と結果

- 対象テスト 7 本を実行し、すべて成功: `consistency-gate.test.js` 15件、`consistency-gate-ui.test.js`、`needs-gate-integration.test.js` 8件、`needs-diagnosis.test.js` 12件、`needs-command-failure.test.js` 7件、`command-receipt.test.js` 7件、`overview-ui.test.js`。
- 対象ファイル限定 ESLint は成功: `project.js`、`toolconfig.js`、agent-project の `ipc.js` / `preload.js`、`renderer.js`、`sections/overview.js`、`sections/needs.js`。
- 全体 `npm run lint` は範囲外の既存問題 `src/features/cowork/main/cowork.js:590` (`cfg` 未使用) 1件で失敗した。対象ファイルには問題なし。
- 初回テストは `yaml` 未導入で開始前に失敗したため、lockfile が無いことを確認後、`npm install --no-package-lock --ignore-scripts` で依存を復元して再実行した。追跡対象ファイルは変更されていない。
- `git status --short --untracked-files=all` と `git diff --check` は出力なし。`tools/agent-dashboard` のソース変更は行っていない。

## 未解決事項・範囲外で見つけた問題

- 未解決: 公式 instance/status 契約に実効 `--config` パスが無いため、自動探索候補と実運用設定の一致は dashboard から保証できない。現 UI はこの制約を明示しており、推測で補わない。
- 未解決: 設定文字列から結線を判定するだけで、codd-gate の存在、互換性、実行成功は確認しない。実測が必要なら agent-project 側の公式契約拡張として別タスクにする。
- 範囲外: 全体 lint の `cowork.js:590` 未使用変数は本タスクでは修正しない。
- 範囲外: agent-project のフック実装、設定注入 CLI の追加、UI からの状態書換は行っていない。

結論: 状態表示の最小既存配置は、全体を `consistencyGateHtml()` の概要セクション、個別失敗を `renderNeedFacts()` の既存「状況」セクションとする現在の構成である。追加コンポーネントや新規 IPC は不要。

# t3 dashboard 結合点調査

## 成果／サマリー

完了条件を「既存実装を変更せず、後続ノードが main 統合後にも保持すべき読み取り・表示の結合点を、公式契約に基づいて確定する」と解釈した。現ブランチには目的の経路がすでに存在し、追加実装は不要だった。リポジトリ内ファイルは変更していない。

### 1. 概要・プロジェクト情報のデータ経路

1. renderer の `reloadProject()` が `api.readProject(state.selectedDir)` を呼ぶ。
2. preload の `readProject` が IPC `dashboard:project` へ渡す。
3. agent-project feature の IPC が `project.readProject(dir, loadConfig())` を呼ぶ。
4. `readProject(workspaceDir, cfg)` は、状態データを `resolveProjectRoot(workspace)` が返すプロジェクトルートから読み、設定はワークスペースを基準に別途 `readToolConfig('agent-project', [workspace, ...agentDirCandidates(workspace)])` で読む。
5. 返却スナップショットの `consistencyGate` が renderer の `state.project` に入り、`renderAllTabs()` → `renderOverview()` → `consistencyGateHtml(p)` で概要に表示される。

結合点:

- `tools/agent-dashboard/src/features/agent-project/main/project.js`
  - `consistencyGateStatus(cfg)`（約 1610 行）: 設定値から表示モデルを一度だけ生成する正規の境界。
  - `readProject()` の返却値 `consistencyGate: consistencyGateStatus(projectCfg)`（約 1767 行）: main→renderer の唯一の追加ペイロード。
- `tools/agent-dashboard/src/renderer/renderer.js`
  - `consistencyGateHtml(p)`（約 1090 行）: 判定はせず表示だけを担当。
  - `bindConsistencyGate(root)`（約 1187 行）: `api.openPath` で設定ファイルを OS の既定エディタに開くだけ。
- `tools/agent-dashboard/src/renderer/sections/overview.js`
  - `renderOverview()` 内の `${consistencyGateHtml(p)}`（約 349 行）: 既存の概要レイアウトへ足す結合点。

### 2. `regression_cmd` / `intake_cmd` の取得元と判定

- 探索順は `readToolConfig()` の first-wins。候補は `[workspace, workspace/.agents, workspace/.agent, ~/.agents（旧環境では ~/.agent）]` 相当で、各候補内は `yaml → yml → json`。
- ワークスペース設定があればグローバル実効設定より優先し、無ければ `~/.agents` の実効設定を使う。
- `resolveProjectRoot()` の `root` 解決だけは、グローバル設定で全ワークスペースが同じ状態先になるのを避けるため `_configFromWorkspace` でローカル設定に限定する。一方、ゲート表示は本体の実効設定と揃えるためグローバル fallback を許す。この差を維持する。
- `consistencyGateStatus` の返却契約:
  - `configFile`
  - `regressionConfigured` / `intakeConfigured`
  - `regressionWired` / `intakeWired`
  - `wired`（両方の AND。renderer で再計算しない）
  - `regressionCmd` / `intakeCmd`（別コマンド設定済みも隠さない）
- 結線判定はキーの有無ではなく、`tools/agent-project/codd_gate_wiring.py` と同じ語順条件:
  - regression: `codd-gate ... verify ... --base`
  - intake: `codd-gate ... tasks ... --debt`
  - `--repos` の有無は問わない。`make -s smoke` 等、汎用フックに置かれた別コマンドは「設定あり・未結線」。
- dashboard は設定文字列だけを読み、codd-gate の実在・互換性・実行成功を検査しない。

### 3. 既存 UI と README の有効化導線

- 概要は「結線済み／一部結線／未結線」の 3 値見出しと、各キーの「設定あり/なし」「結線済み/未結線」「現在のコマンド」を表示する。
- 未結線キーだけ、`tools/agent-dashboard/README.md`「一貫性ゲート」と同じ設定行を表示する:
  - `regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'`
  - `intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'`
- `regression_cmd` のみ、ソース版 `tools/agent-project/` で `python3 codd_gate_regression.py --config /path/to/.agents/agent-project.yaml` を使える。既存設定ファイルが無い場合は CLI を勧めない。
- `intake_cmd` の注入 CLI は無いため、設定ファイルを直接編集する案内が正。
- 「設定ファイルを開く」は `api.openPath` のみで、dashboard 自身は設定を書き換えない。これは done 不変条件と、UI が第二の状態書き手にならない境界を守る。

### 4. needs-diagnosis と codd-gate／回帰失敗の表示経路

producer→viewer の経路:

1. agent-project の `agent_project/needs.py` が `needs/<id>.md` を MADR frontmatter 付きで生成する。失敗情報の公式構造化キーは `failure-phase`, `verify-verdict`, `failure-summary`, `failure-resolution`, `failure-category`, `failure-owner`, `failure-command`, `failure-workdir`, `failure-exit`, `failure-target` 等。
2. 回帰ゲート失敗時は `agent_project/mr.py` が `regression_cmd` を `cfg.workdir` で実行し、非 0 なら `_block(...)` して done を確定せず、理由へ `回帰検知: グローバル検査 \`<実コマンド>\` 失敗` を残す。
3. dashboard main の `parseNeeds()` が構造化 frontmatter を `failureSummary` / `failureResolution` / `failureContext` / `failurePhase` 等へ運ぶ。新規票は producer の解釈をそのまま使い、`_diagnoseFailure()` は旧票だけの fallback。
4. `readProject()` が `needs` 配列をスナップショットへ載せる。needs ファイル欠落時は backlog の `proposed/blocked/review` を正として表示用カードを合成するが、状態ファイルは書かない。
5. renderer `renderNeeds()` → `renderNeedDetail()` → `renderNeedFacts(p,n)` が既存の「検証失敗」「確認・対処」「実行条件」を表示する。
6. `needGateSource(failure,n,gate)` が codd-gate 由来だけを `regression` または `verify` と分類し、`renderNeedFacts` が同じカードへ「一貫性ゲート」節を追加する。

由来判定の重要点:

- `failurePhase === 'regression'` または旧票の「回帰検知」に加え、記録された summary/command に実際の `codd-gate` が含まれることを要求する。現在の設定が後から変わっていても、失敗時の記録を根拠にする。
- タスク自身の verify が codd-gate の場合は `verify` 経路として扱い、「regression_cmd が止めた」と誤表示しない。
- 通常の pytest 等の失敗、失敗要約の無い needs、旧 main 由来で `consistencyGate` が無いペイロードにはゲート節を出さない。
- intake 結線済みなら「正常実行時にドリフトを修復タスクへ起票」、未結線なら概要の有効化導線へ誘導する。既存の失敗要約・context・「判断材料を見る」折り畳みは保持する。

### 5. 公式 needs / inbox / commands 契約（変更禁止境界）

- `needs/<id>.md`: `## Decision Outcome`（旧 `## フィードバック` 互換）へ人の指示を書き `[x]` で確定。dashboard の `submitFeedback` はこの契約だけを編集する。
- `inbox/<name>.json`: 新規タスク投入の E4 push 契約。`enqueueToInbox` が task schema の許可フィールドを投函し、本体 `ingest_inbox` が backlog 化する。
- `commands/<name>.json`: 既存タスクへの判断・操作。dashboard は `.tmp` 書込み→rename で原子的に drop し、本体 `ingest_commands` が CLI と同一ロジック・同一 DR で処理する。失敗は `.err` へ退避され、dashboard は `listCommandFailures` で対応 needs に表示する。
- dashboard が backlog/project/status や非公式状態ファイルを直接書き換える経路は追加しない。codd-gate は外部 CLI の公式 E2 (`regression_cmd`) / E3 (`intake_cmd`) と共通 task/repos schema だけで接続する。

### 6. 後続ノードが保持すべき最小統合点

- t2 の競合方針どおり、main の `resolveProjectRoot`、state-worktree 解決、`_busRecency`、最新 needs キャッシュ署名を正とし、一貫性ゲート差分だけを併存させる。
- `project.js`: main の構造へ `consistencyGateStatus` と `readProject()` の `consistencyGate` 返却を残す。設定値の取得は `projectCfg` を再利用し、別設定ファイルや専用状態を増やさない。
- `renderer.js` + `overview.js`: `consistencyGateHtml` / `bindConsistencyGate` と概要への 1 箇所の挿入を保持する。タブ追加や新しい設定保存 API は不要。
- `needs.js`: main の `p.needs` 全体キャッシュ署名を保持し、別要素 `p.consistencyGate || null` を追加する。`needGateSource` と `renderNeedFacts` の追加ブロックを保持する。
- README: 設定編集、sibling CLI は regression のみ、intake は直接編集、読み取り専用という導線を UI と同期させる。
- テスト: `consistency-gate.test.js`, `consistency-gate-ui.test.js`, `needs-gate-integration.test.js` を package の test 列へ残し、main 側の既存テストも維持する。

## 検証内容と結果

- `tools/agent-dashboard` で `npm test` を実行: exit 0、全テスト成功。
- 対象の `consistency-gate.test.js`: 10 passed。
- `consistency-gate-ui.test.js`: ok。
- `needs-gate-integration.test.js`: 8 passed。
- 既存 UI 回帰対象 `overview-ui.test.js`, `detail-tabs-ui.test.js`, `needs-diagnosis.test.js` を含む全 test が成功。
- 実行後 `git status --short`: clean。リポジトリ内変更なし。
- t2 の `git-handoff.md` と照合し、6 競合候補のうち本件に関係する `project.js` / `needs.js` / `overview-ui.test.js` の最小解消方針と矛盾しないことを確認した。

## 採用した前提・未解決事項・範囲外

- 前提: このノードは調査・結合点確定が目的であり、現ブランチに対象実装とテストが存在するため、コード変更ではなく自己完結した handoff が成果物となる。
- 前提: 「既存 README」は dashboard README と、その有効化手順の正である agent-project README の双方を指す。
- 未解決: t2 記載の main への実 rebase/競合解消は agent-flow の統合工程に残る。本ノードでは禁止された rebase/commit/push を行っていない。
- 範囲外: agent-project 本体のフック実装、done 不変条件の変更、UI からの設定・状態書換、非公式状態ファイルの追加は行っていない。

@followup agent-flow の統合工程では、t2 の 6 ファイル解消方針と本書 §6 を同時に適用し、main 上で lint と全 test を再実行する。

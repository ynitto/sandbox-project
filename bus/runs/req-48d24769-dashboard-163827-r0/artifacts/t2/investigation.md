# agent-dashboard 読取経路調査

## 完了条件と採用した前提

- 完了条件は、`tools/agent-dashboard/src/features/agent-project/main/project.js` における
  `regression_cmd`、`intake_cmd`、`needs`、`commands` の読取元・変換・renderer までの
  受け渡しを特定し、追加ストレージや新規 IPC を作らず再利用できる供給点を示すこと。
- このノードは調査タスクと解釈した。現行コードに必要な供給契約が既に存在したため、
  `tools/agent-dashboard` のコードは変更していない。
- 「commands の読取経路」は `project.js` が画面用スナップショットへ取り込む経路を対象とし、
  `actions.js` の書込み経路や agent-project 本体の取り込み実装は参照確認だけとした。

## 成果／結論

再利用すべき供給点は `readProject(workspaceDir, cfg)` の返す既存プロジェクトスナップショットである。
main 側で読み取り・判定を一度だけ行い、既存の
`dashboard:project` IPC → preload `readProject()` → renderer `reloadProject()` →
`state.project` へそのまま供給できる。新規 IPC、非公式キャッシュ、状態書換えは不要。

### 1. regression_cmd / intake_cmd

読取経路:

1. `readProject()` が workspace と `.agent` / `.agents` 候補を
   `readToolConfig('agent-project', ...)` へ渡す。
2. `toolconfig.js` は `agent-project.yaml` → `.yml` → `.json` の順で最初の設定を読む。
   YAML はトップレベルのスカラだけ、JSON はトップレベル object を `values` として返す。
3. `_configFromWorkspace()` が、見つかった設定が当該 workspace 配下にあることを確認する。
   `readToolConfig()` のグローバル設定フォールバックは結線表示には採用しない。
4. `consistencyGateStatus()` が `values.regression_cmd` / `values.intake_cmd` を読み、
   空白を除去した値と結線フラグを作る。
5. `readProject()` の返り値 `consistencyGate` として供給する。

既存ペイロード:

```text
project.consistencyGate = {
  configFile: string | null,
  regressionWired: boolean,
  intakeWired: boolean,
  wired: boolean,
  regressionCmd: string | null,
  intakeCmd: string | null
}
```

結線判定は agent-project 本体の `codd_gate_wiring.py` と同じ語順契約を再利用している。

- regression: `codd-gate ... verify ... --base`
- intake: `codd-gate ... tasks ... --debt`

したがって、単にキーが非空かでは判定しない。`regression_cmd: make -s smoke` のような
正当な別ゲートは値を表示できるが、codd-gate 結線済みにはしない。

### 2. needs

読取経路:

1. `resolveProjectRoot(workspace)` で公式状態ルートを解決する。
2. `listMdDir(<root>/needs, parseNeeds)` で `needs/*.md` を読む。
3. `parseNeeds()` が MADR frontmatter、本文、Decision Outcome の `[x]`、
   判断材料、失敗情報を表示モデルへ変換する。
4. `synthesizeNeedsFromBacklog()` が、票のない `proposed` / `blocked` / `review`
   タスクを backlog status から表示専用に合成する。
5. `attachDeliveryHintsFromBacklog()` が MR / delivery 情報を補う。
6. `readProject()` の `needs` 配列として供給する。

再利用すべき公式契約は agent-project の `needs.py` が書く frontmatter である。
特に失敗表示は `failure-summary`、`failure-resolution`、`failure-category`、
`failure-owner`、`failure-command`、`failure-workdir`、`failure-exit`、
`failure-target`、`failure-class`、`failure-chain`、`failure-phase`、
`verify-verdict` を `parseNeeds()` が既に構造化している。
旧票だけ `_diagnoseFailure()` の散文解析へフォールバックする。

`needs` は独立した真実ではなく backlog status の投影、という本体契約も保たれている。
票欠落時の合成は表示専用でファイルを書かないため、done 不変条件を変更しない。

### 3. commands

`project.js` が読む commands は次の二系統に限定される。

- `commands/*.err`: `listCommandFailures()` が公式の
  `{error, failed_at, command:{command,id,...}}` を読み、task id ごとの最新一件を
  未決着 need の `need.commandFailure` へ付与する。この形式は agent-project 本体の
  `_reject_command()` が生成する契約と一致する。
- 未処理 replan: `replanRequestPending()` が `.replan.request`、または
  `commands/*.json` の `{command:"replan"}` を読み、`project.replanPending` を返す。

通常の未処理 `approve` / `hold` / `pin` / `defer` / `revise` 等を一覧化する読取契約は
`project.js` にはない。ゲート失敗の意味を出す用途では、新規 commands スキャンを足さず、
既存の `needs[].commandFailure` と構造化失敗情報を使うのが最小かつ公式契約に沿う。

### 4. renderer への既存供給経路

```text
project.readProject()
  -> IPC "dashboard:project"
  -> preload api.readProject(dir)
  -> renderer reloadProject()
  -> state.project
```

概要は `state.project.consistencyGate`、要対応は
`state.project.needs[]` の `failureSummary` / `failureResolution` /
`failureContext` / `commandFailure` を読むことで、同一スナップショット内の事実を使える。

## 検証

- `node test/consistency-gate.test.js`: 6 passed
- `node test/needs-diagnosis.test.js`: 13 passed
- `node test/needs-command-failure.test.js`: 4 passed
- `git status --short`: 変更なし

上記で、設定探索・結線判定・グローバル設定除外、needs の構造化失敗／旧票フォールバック、
commands `.err` の最新一件集約を確認した。

## 未解決事項・範囲外で見つけた点

- YAML リーダーはトップレベルの単行スカラだけを読む簡易契約であり、複数行 YAML scalar は対象外。
  現在の agent-project 設定契約と対象キーには十分で、今回の変更対象にはしない。
- `intake_cmd` 用の注入 CLI は本体にも存在しない。README 契約どおり設定ファイルを人が編集する。
- agent-project 本体のフック実装、done 判定、commands/needs への書込みは変更していない。

@followup UI 実装では `readProject()` の既存 `consistencyGate` と `needs` を直接使い、
有効化は設定ファイルを開く／README と同じ sibling CLI の提示までに留める。

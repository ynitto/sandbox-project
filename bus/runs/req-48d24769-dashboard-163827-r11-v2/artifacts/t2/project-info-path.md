# project.js 調査結果: regression_cmd / intake_cmd のプロジェクト情報経路

## 完了条件と採用した前提

- 完了条件は、`regression_cmd` / `intake_cmd` が dashboard のプロジェクト情報へ入る現行経路、UI 向けデータ契約、欠損時の表現、後続実装が再利用すべき結合点を特定することとした。
- 「概要・プロジェクト情報生成記号」は、一覧用 `discover()` ではなく、概要画面が受け取る完全スナップショット `readProject()` と解釈した。現状、一覧の project summary にはゲート情報がなく、選択後の project payload にだけ存在する。
- 調査タスクのためコード変更は不要と判断した。`renderer.js`、状態変更処理、agent-project 本体のフック実装は変更していない。

## 成果・サマリー

現行経路は次の一本に集約されている。

```text
window.api.readProject(workspace)
  -> preload: invoke('dashboard:project', { dir })
  -> ipc: project.readProject(dir, loadConfig())
  -> readProject(): readToolConfig('agent-project', [workspace, workspace/.agents, workspace/.agent])
  -> consistencyGateStatus(projectCfg)
  -> project payload の consistencyGate
  -> 概要表示 / needs のゲート失敗補足
```

根拠:

- `src/features/agent-project/preload.js:12`: `readProject` の IPC 表面。
- `src/features/agent-project/main/ipc.js:57-60`: `dashboard:project` が `project.readProject()` を返す。
- `src/features/agent-project/main/project.js:1949`: agent-project 設定を一度読む。
- `src/features/agent-project/main/project.js:2001`: 完全スナップショットへ `consistencyGate` として結合する。
- `src/features/agent-project/main/project.js:1793-1819`: コマンド値と判定を UI 向けに正規化する唯一の関数。

設定探索は `readToolConfig()` の既存経路を再利用しており、渡されたディレクトリごとに `yaml -> yml -> json`、ディレクトリは `workspace -> workspace/.agents -> workspace/.agent -> agentHomeDir()` の順で、最初に見つかったファイルを採る。YAML はトップレベルのスカラのみ、JSON はトップレベル object を読む。ローカル JSON が壊れている場合は global 設定へフォールバックしない。

UI 向け `consistencyGate` の形は以下。

```js
{
  configFile: string | null,
  configError: string | null,
  configSource: 'dashboard-auto-discovery',
  explicitConfigUnknown: true,
  regressionConfigured: boolean,
  intakeConfigured: boolean,
  regressionWired: boolean,
  intakeWired: boolean,
  wired: boolean,
  regressionCmd: string | null,
  intakeCmd: string | null,
}
```

意味は次のとおり。

- `*Configured`: 空でないコマンド値が設定されているか。
- `*Wired`: 値が codd-gate の正規形に合うか。回帰は `codd-gate ... verify ... --base`、取り込みは `codd-gate ... tasks ... --debt` の同一行・語順マッチ。
- `wired`: `regressionWired && intakeWired`。概要と needs が個別に再計算しないための main 側の派生値。
- `*Cmd`: codd-gate 以外の汎用フックでも値を保持する。これにより UI は「未設定」と「別コマンド設定済み」を区別できる。

欠損・異常時の表現:

| 状況 | Cmd | Configured | Wired | 補足 |
|---|---|---:|---:|---|
| 設定ファイルなし | `null` | `false` | `false` | `configFile: null`、ただし `consistencyGate` object 自体は常に返る |
| キーなし / YAML null / 空文字 / 空白のみ | `null` | `false` | `false` | 空白は trim 後に欠損化 |
| codd-gate 以外のコマンド | 元の文字列 | `true` | `false` | 汎用フック設定を隠さない |
| 片方だけ結線 | 片方は値、他方は `null` | 個別判定 | 個別判定 | 全体 `wired: false` |
| 両方結線 | 両方とも値 | `true` | `true` | 全体 `wired: true` |
| 不正なローカル JSON | `null` | `false` | `false` | `configFile` は不正ファイル、`configError: 'invalid-json'` |

再利用可能な結合点は `readProject()` の `consistencyGate` プロパティである。後続 UI は設定を再読込・再判定せず `p.consistencyGate` をそのまま使うのが最小かつ一貫する。判定ロジックを増やす場合も `consistencyGateStatus()` 内に留め、概要と needs の両利用箇所へ同じ payload を流す。`discover()` の一覧 summary へ載せる要求は現状ないため追加しない。

## 検証内容と結果

- `node test/consistency-gate-ui.test.js`: `consistency-gate-ui: ok`。
- `node --check src/features/agent-project/main/project.js`: 成功。
- `node test/consistency-gate.test.js`: 実行環境に package.json 記載の `yaml` が未インストールで、ロード時に `MODULE_NOT_FOUND`。依存導入による調査 worktree の変更は避けた。代替として既存テストケースと実装の静的突合、および構文検査を行った。
- `git status --short`: 出力なし。対象 worktree に変更なし。
- 静的突合: preload -> IPC -> `readProject()` -> `consistencyGateStatus()` -> project payload の経路をソース上で確認した。

## 未解決事項・範囲外で見つけた点

- agent-project 本体が `--config` で明示した実効設定パスは instance/status 契約に載らない。dashboard は自動探索しかできないため、`explicitConfigUnknown: true` が常に付き、本体の実効設定と食い違う可能性が残る。
- `regression_cmd` / `intake_cmd` は codd-gate 専用キーではない。`Configured` と `Wired` を混同してはいけない。
- UI から設定や状態を書き換える結合点は作らない。未結線時の有効化は設定ファイル編集または README 記載の sibling CLI 導線に留める。

@followup agent-project 本体の実効 `--config` パスを正確に表示する必要がある場合は、instance/status の読み取り専用契約へ設定ソースを追加する別タスクとして扱う。

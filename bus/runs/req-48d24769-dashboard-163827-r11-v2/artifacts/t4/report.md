# tools/agent-dashboard 公式ゲート境界・有効化手順の確認

## 成果／サマリー

追加変更は不要と判断した。対象ブランチの `tools/agent-dashboard` には、公式契約だけで一貫性ゲートの状態と失敗の意味を表示し、未結線時に README 準拠の有効化手順を案内する実装・README・テストが既に揃っている。作業ツリーの差分は 0 件。

公式の読み取り境界は次のとおり。

- 結線状態: dashboard が自動探索した `agent-project.yaml|yml|json` のトップレベル `regression_cmd` / `intake_cmd` の文字列だけを読む。`codd-gate verify --base` と `codd-gate tasks --debt` の語順で結線を判定し、コマンドの実在・互換性・実行成否は推測しない。
- 失敗の意味: `needs/<id>.md` の公式 `failure-*` frontmatter（phase / summary / command 等）を主入力にする。旧記録だけは既存の散文解析へフォールバックする。codd-gate の記録が確認できる場合だけ「一貫性ゲート由来」と表示し、`regression` 失敗なら done 確定前に止めたことを示す。
- `inbox/`: 公式の取り込み待ちファイルを列挙するだけで、ゲート結線の根拠にはしない。
- `commands/`: 公式の `.err` と `processed/*.json` を操作失敗／受理レシートとして読むだけで、ゲート結線や done を独自に書き換えない。
- dashboard は設定・タスク状態・done を変更せず、設定候補を OS エディタで開く導線だけを提供する。

未結線時に提示する README 準拠手順は次のとおり。

```yaml
regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
```

`<root>` は対象プロジェクトのルートへ置換する。既存 JSON 設定ではトップレベル object の該当プロパティだけを追加・置換し、ファイル全体を上書きしない。既存 YAML があり `regression_cmd` が未設定なら、ソースの `tools/agent-project/` で sibling CLI を実行できる。

```bash
python3 codd_gate_regression.py --config /path/to/.agents/agent-project.yaml
```

この CLI が更新するのは `regression_cmd` のみ。`intake_cmd` は設定ファイルへ直接記入する。`--dry-run` なら書込みなしで結果を確認できる。設定ファイル未検出、JSON 設定、既存の別 `regression_cmd` がある場合は CLI を安易に勧めない表示になっている。

## 検証内容と結果

- `npm test`: PASS（agent-dashboard 全テスト）。
- ゲート関連の再実行: PASS。
  - `consistency-gate.test.js`: 15/15
  - `consistency-gate-ui.test.js`: PASS
  - `needs-gate-integration.test.js`: 8/8
  - `needs-diagnosis.test.js`: 12/12
  - `needs-command-failure.test.js`: 7/7
  - `command-receipt.test.js`: 7/7
- 関連ファイル限定 ESLint: PASS。
- `git status --short` / `git diff -- tools/agent-dashboard`: 差分なし。
- 全体 `npm run lint`: FAIL。今回の対象外かつ既存の 4 件（`actions.js:373`、`agent.js:20`、`cowork.js:590`、`backlog.js:1210`）で unused/no-useless-assignment。ゲート関連ファイルには lint 違反なし。

## 採用した前提・未解決事項・範囲外

- 完了条件を「公式設定・needs/inbox/commands のみから判断でき、未結線時の設定編集と sibling CLI 手順が README と一致することの確認」と解釈した。現物が既に満たすため、重複実装や README 改稿は行わなかった。
- dashboard の自動探索設定は、agent-project が起動時に `--config` で選んだ実効設定と一致するとは限らない。instance/status 契約に解決済み設定パスが無いため、画面もその制約を明示して断定しない。
- codd-gate の実在・バージョン/schema 互換・実行成功は設定文字列だけでは判断しない。確認する場合は agent-project README の読み取り専用 `codd_gate_wiring.py` が正準手順。
- `agent-project` のフック追加、UI からの done／状態書換、非公式ファイルへの書込み、README 改稿は範囲外として未実施。
- @followup 全体 lint の既存 4 エラーは別タスクで整理可能。

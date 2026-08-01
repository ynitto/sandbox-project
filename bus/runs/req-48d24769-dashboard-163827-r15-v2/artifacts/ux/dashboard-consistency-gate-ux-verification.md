# dashboard 一貫性ゲート UX 検証

## 判定

verify=pass

重大な要求不充足、誤表示、破壊的な状態書換は確認しなかった。

## 独立検算

- 差分は `tools/agent-dashboard` 配下の5ファイルだけ（227 insertions / 9 deletions）。スコープ外変更なし。
- 概要は全体の3状態（結線済み／一部結線／未結線）を先に示し、その下で `regression_cmd` / `intake_cmd` ごとに設定有無・結線・現在値を分ける。現在状態の把握から詳細へ降りる情報階層になっている。
- 技術情報欄へ重複掲載せず、概要を現在状態の正本、needs を失敗時点の個別事実として分離している。現在設定から過去の失敗原因を推測しない。
- 未結線時は未結線キーだけに設定例を出し、YAML / JSON、既存値の置換警告、設定ファイル未検出を分ける。設定例と sibling CLI は `tools/agent-project/README.md` の正式手順と一致する。
- needs は「検証失敗の要約 → 確認・対処 → 実行条件 → 一貫性ゲート由来」の順。`failure-*` の既存要約・context・詳細折り畳みを保持し、regression とタスク自身の verify を区別する。
- 新しい保存・approve・complete・status 更新経路はない。有効化ボタンは既存 `openPath` で設定候補を開くだけで、描画モデル、`kind=blocked`、`decided=false` を変更しない。

## 実行結果

- `node test/needs-gate-integration.test.js`: 10 passed
- `node test/consistency-gate.test.js`: 15 passed
- `node test/consistency-gate-ui.test.js`: success
- `node test/needs-diagnosis.test.js`: 12 passed
- `npx eslint test/needs-gate-integration.test.js src/renderer/renderer.js`: success
- `npm test`: success
- `git diff --check main...HEAD -- tools/agent-dashboard`: success
- `npm run lint`: 既存の `src/features/cowork/main/cowork.js:590` の未使用変数1件だけで失敗。今回の差分外。

## issues

- (minor) `CONSISTENCY-GATE-DESIGN.md:122-133` は run 内部名 `t3` / `t4`、現在は解消済みの CLI プレースホルダ不一致、完了済み `@followup`、最終差分と一致しない「設計書以外のファイルは変更していない」を残している。製品設計書として残すなら、最終状態へ更新する。
- (minor) `test/consistency-gate-ui.test.js:249-252` の README 同期テストは README 側の CLI 名しか抽出せず、`<状態 clone>/agent-project.yaml` は画面側の固定文字列だけを検査している。README の `--config` 引数を抽出し、HTML 側と比較する。

## agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---:|---:|---:|---:|
| functional | LGTM | 0 | 0 | 1 |
| ai-antipattern | LGTM | 0 | 0 | 1 |
| architecture | LGTM | 0 | 0 | 0 |
| test | LGTM | 0 | 0 | 1 |

```json
{"ok":true,"issues":["(minor) tools/agent-dashboard/CONSISTENCY-GATE-DESIGN.md:122-133 の検証記録・未解決事項が最終差分に追随していない。t3/t4、解消済み CLI 不一致、完了済み @followup、変更範囲の記述を最終状態へ更新する。","(minor) tools/agent-dashboard/test/consistency-gate-ui.test.js:249-252 は README の --config プレースホルダを実際には比較していない。README の CLI ブロックから引数を抽出して画面表示と比較する。"]}
```

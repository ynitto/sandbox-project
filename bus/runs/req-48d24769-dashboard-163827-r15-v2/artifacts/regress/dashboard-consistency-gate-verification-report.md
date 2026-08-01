# t9 までの dashboard 一貫性ゲート変更 — 独立検証

## 判定

`verify=pass`

t9 の追加テストと関連テスト、リポジトリ所定の全テスト、対象 ESLint、diff check は成功した。全 lint は未変更の `src/features/cowork/main/cowork.js:590` にある既存の未使用変数 `cfg` 1件で失敗したが、今回差分による回帰ではないため minor と判定した。最終検証出力に `ok:false` は無かった。

## 最終検証結果（すべて新規プロセス）

| 検証 | exit code | terminal ok | `ok:false` | 結果 |
|---|---:|---|---|---|
| `node test/needs-gate-integration.test.js` | 0 | true | なし | 10 passed |
| `node test/consistency-gate.test.js` | 0 | true | なし | 15 passed |
| `node test/consistency-gate-ui.test.js` | 0 | true | なし | `consistency-gate-ui: ok` |
| `node test/needs-diagnosis.test.js` | 0 | true | なし | 12 passed |
| `npm test` | 0 | true | なし | 全テスト成功 |
| `npx eslint test/needs-gate-integration.test.js` | 0 | true | なし | 問題なし |
| `npm run lint` | 1 | true | なし | 既存エラー1件 |
| `git diff --check main...HEAD` | 0 | true | なし | 問題なし |

`terminal ok` はプロセスが終了し exit code を取得できたこととして検査した。`ok:false` は stdout/stderr を機械走査し、検出時は exit code にかかわらず失敗とする条件で確認した。

## 再導出・差分照合

- HEAD は t9 成果記載どおり `4b5b626399e97f34a987429cce0c761d8d38c8e4`。
- t9 単体差分は `test/needs-gate-integration.test.js` のみ、追加82行・削除0行。
- `main...HEAD` の run 全体差分は5ファイル、追加227行・削除9行。全ファイルが `tools/agent-dashboard/` 配下で、スコープ外変更はない。
- 追加2テストを抜き取り確認し、3結線状態、公式 `consistencyGate` payload、設定ファイルを開くだけの操作、公式 needs frontmatter、codd-gate 回帰失敗要約、`kind=blocked` / `decided=false` を検査していることを確認した。
- production 経路も抜き取り確認し、`readProject().consistencyGate` と `parseNeeds()` の構造化値を表示に使い、有効化ボタンは `api.openPath()` のみを呼ぶことを確認した。
- t9 報告の関連テスト件数（10、15、12）と再実行結果は一致した。重複・欠落した変更ファイルは見つからなかった。

## 検証環境の準備

初回は依存未導入のため `yaml` / ESLint が見つからず検証コード到達前に失敗した。lockfile が無いため `npm ci` は exit 1。その後 `npm install --no-package-lock --no-audit --no-fund` を exit 0、terminal ok=true、`ok:false` なしで実行し、ソースや package metadata を変更せず依存を準備して上表を再実行した。

## Issues

- (minor) `tools/agent-dashboard/src/features/cowork/main/cowork.js:590` の `cfg` が未使用で、`npm run lint` は exit 1。対象行は `main` と同一で今回差分外。別タスクで、利用予定がなければ宣言を削除する。

```json
{"ok": true, "issues": ["(minor) tools/agent-dashboard/src/features/cowork/main/cowork.js:590 の未変更の cfg が未使用なため npm run lint は exit 1。利用予定がなければ別タスクで宣言を削除する"]}
```

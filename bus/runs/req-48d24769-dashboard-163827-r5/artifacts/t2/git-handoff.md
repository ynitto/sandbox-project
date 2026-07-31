# t2 Git handoff

## 成果／サマリー

- 完了条件を「`origin/main` と dashboard 実装の意図を照合し、競合箇所ごとの最小解消方針を確定する。履歴操作は agent-flow に委ねる」と解釈した。
- `origin/main` と HEAD の 3-way 統合を読み取り専用で再現し、実競合候補を次の 6 ファイルに限定した。
  - `tools/agent-dashboard/package.json`
  - `tools/agent-dashboard/src/features/agent-project/main/project.js`
  - `tools/agent-dashboard/src/features/agent-project/main/toolconfig.js`
  - `tools/agent-dashboard/src/renderer/sections/needs.js`
  - `tools/agent-dashboard/test/detail-tabs-ui.test.js`
  - `tools/agent-dashboard/test/overview-ui.test.js`
- 解消方針は以下。いずれも main の現構造を正とし、一貫性ゲート差分だけを追加する。
  - `package.json`: main の `lint` と最新 test 列を保持し、新規 3 テスト（`consistency-gate.test.js`、`consistency-gate-ui.test.js`、`needs-gate-integration.test.js`）を test 列へ追加する。
  - `project.js`: main の `resolveProjectRoot`／state-repo 解決、`_busRecency`、`projectLiveness(dir, undefined, cfg)`、`nodes` を保持する。`consistencyGateStatus` は `_busRecency` と併存させ、`readProject` の返却値へ `consistencyGate: consistencyGateStatus(projectCfg)` のみ追加する。旧 `root`／`state_branch`／`toStateWorktree` 解決は復活させない。
  - `toolconfig.js`: main の `parseYaml` + `isPlainObject` + `scalarString` 実装を採用する。ブランチ側の手書き YAML parser は捨てる（folded/literal scalar は main の YAML parser が扱う）。
  - `needs.js`: main の `p.needs` 全体によるキャッシュ署名を保持し、別要素として `p.consistencyGate || null` を追加する。旧の部分配列署名へ戻さない。
  - `detail-tabs-ui.test.js`: main で追加された全引数・stub を保持する。一貫性ゲート側はこの競合ブロックに追加要件がないため main 側を採用する。
  - `overview-ui.test.js`: main の `nodesSummaryHtml` テストとブランチの一貫性ゲートテストを順番に両方残す。
- リポジトリ内ファイルは変更していない。rebase／commit／push／checkout／branch／stash は実行していない。

## 検証内容と結果

- `git merge-tree <merge-base> origin/main HEAD` により、履歴を書き換えず 3-way 統合を再現した。競合マーカーが生じるファイルは上記 6 件のみ。
- 現 HEAD で次を実行し、すべて成功した。
  - `node test/consistency-gate.test.js`: 10 passed
  - `node test/consistency-gate-ui.test.js`: ok
  - `node test/needs-gate-integration.test.js`: 8 passed
  - `node test/detail-tabs-ui.test.js`: all tests passed
  - `node test/overview-ui.test.js`: all tests passed
- `npm run lint` は現 HEAD の `package.json` に `lint` script が無いため実行不可（`Missing script: lint`）。main 側には script があり、統合時は保持する方針。
- 最終 `git status --porcelain=v2 --branch`: detached HEAD `7a0e5d18...`、作業ツリー／index とも clean。

## 前提・未解決事項・範囲外

- 前提: 後段の具体的な Git 利用規約（`rebase` 禁止）が冒頭の「リベースを完了する」より優先される。そのため、この worker が安全に完了できる範囲は競合の再現・解消方針確定・既存実装の検証まで。
- 未解決: 実際の rebase 完了と競合ファイルへの解消反映は、rebase を許可された agent-flow の統合手順で上記方針どおり行う必要がある。現在は rebase 状態ではないため、この worktree へ解消済みファイルを書けば main の大量変更を通常差分として混入させることになり、最小差分・範囲厳守に反する。
- 範囲外の問題は変更していない。agent-project 本体のフック追加、UI からの状態書換、push は行っていない。

@followup agent-flow の統合工程で rebase を開始し、上記 6 ファイルだけを記載方針で解消してから main 上で lint と全 test を再実行する。

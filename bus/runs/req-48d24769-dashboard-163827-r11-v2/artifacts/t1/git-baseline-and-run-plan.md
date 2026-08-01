# Git baseline and new-run plan

## 完了条件と採用前提

- このタスクは調査・計画のみとし、指定 worktree のソース、index、branch は変更しない。
- 「最新 main 統合」は依存タスクが明示した target revision `3756b4f07cf3d0c9c6225cd1f7f6092ff27dc9f2`（同時に `origin/main`）に対して判定する。
- 一方、確認時点のローカル `refs/heads/main` は `45bb579dfe6621146162fd9e2442575f955f132c` まで進んでいるため、両者を区別して記録する。
- 競合とは Git index の unmerged entry、または merge の content conflict を指す。Markdown の末尾空白だけでは競合と判定しない。
- 旧 run の `done` は一切継承せず、本記録を新規 run の再検証起点とする。

## 成果／サマリー

### 1. 59ccf49e の位置づけ

- worktree HEAD: `59ccf49e71d41fe35533ae718f010649863bd6a3`（detached）
- commit: `[agent-flow] base-sync (req-48d24769-dashboard-163827-r11-v1)`
- 第1親: `635bc8d17c339fa6d29a9f2efd9934bc09ab8d42`
- 第2親: `3756b4f07cf3d0c9c6225cd1f7f6092ff27dc9f2`
- `origin/ap/dashboard-163827` も `59ccf49e` を指す。

したがって、`59ccf49e` は依存タスクが示した target main `3756b4f` を直接第2親として統合した起点であることを確認した。

ただし、ローカル `main` はその後 `45bb579` へ1 commit進んでいる。`main...HEAD` は `main` 側1 commit、HEAD側25 commits。追加の main commit は `docs/plans/2026-08-01-agent-dashboard-selective-note-tasking-design.md` だけを追加しており、許可範囲 `tools/agent-dashboard` とは重ならない。`merge-tree` による read-only 予行では、このファイル追加だけで新規競合はなかった。

### 2. 59ccf49e で解決された6ファイルの content conflict

`git show --remerge-diff 59ccf49e` の `remerge CONFLICT (content)` から次の6ファイルを再導出した。

1. `tools/agent-dashboard/package.json`
2. `tools/agent-dashboard/src/features/agent-project/main/project.js`
3. `tools/agent-dashboard/src/features/agent-project/main/toolconfig.js`
4. `tools/agent-dashboard/src/renderer/sections/needs.js`
5. `tools/agent-dashboard/test/detail-tabs-ui.test.js`
6. `tools/agent-dashboard/test/overview-ui.test.js`

remerge diff には解決時の追加調整として次の3ファイルも現れるが、これらに `remerge CONFLICT` は付かないため、6競合の数には含めない。

- `tools/agent-dashboard/test/consistency-gate-ui.test.js`
- `tools/agent-dashboard/test/needs-command-failure.test.js`
- `tools/agent-dashboard/test/needs-gate-integration.test.js`

### 3. 現在の差分

ローカル `main` と HEAD の共通祖先 `3756b4f` から HEAD までの `tools/agent-dashboard` 差分は13ファイル、`+1152/-7`。

- 変更: `README.md`, `package.json`
- 変更: `src/features/agent-project/main/project.js`
- 変更: `src/features/agent-project/main/toolconfig.js`
- 変更: `src/renderer/renderer.js`
- 変更: `src/renderer/sections/needs.js`
- 変更: `src/renderer/sections/overview.js`
- 追加: `test/consistency-gate-ui.test.js`
- 追加: `test/consistency-gate.test.js`
- 変更: `test/needs-command-failure.test.js`
- 変更: `test/needs-diagnosis.test.js`
- 追加: `test/needs-gate-integration.test.js`
- 変更: `test/overview-ui.test.js`

### 4. 未解決競合と作業ツリー

- `git status --porcelain=v2 --branch`: detached HEAD の情報のみ。tracked/untracked の変更なし。
- `git diff HEAD` / `git diff --cached HEAD`: いずれも空。
- `git ls-files --unmerged`: 空。index 上の未解決競合は0件。
- `tools/agent-dashboard` の行頭 conflict marker (`<<<<<<<`, `=======`, `>>>>>>>`) 検索: 0件。
- 現在のローカル main を重ねる `git merge-tree` 予行: content conflict 0件。

結論: 59ccf49e の6 content conflict は解決済みで、現作業ツリーに未解決競合はない。

## 新規 run の実行計画

1. **起点を固定する**: 旧 run の done を参照せず、`59ccf49e` と本成果物を入力にする。開始時に HEAD、target main、clean status、unmerged=0 を再確認する。
2. **main の鮮度を明示する**: agent-flow の target が `3756b4f` なら統合済みとして noop。ローカル `main=45bb579` を正典にする run なら、通常の base-sync に任せて1 commit分を統合し、競合0を再確認する。worker 自身は checkout/rebase/merge しない。
3. **既存成果を差分として再評価する**: 上記13ファイルを新規 run の実装候補集合とし、要件・テストを新たに判定する。旧 run の成功判定は継承しない。
4. **競合解決を保護する**: 特に6 conflict ファイルと3追加調整ファイルを変更する後続タスクは、両親の意図（main側の更新と consistency-gate 側の変更）が残ることを確認する。Markdown 末尾空白のみを競合根拠にしない。
5. **範囲を限定する**: 実装書込は `tools/agent-dashboard` のみに限定する。`agent-project` のフック実装や UI からの done 状態書換は行わず、必要なら `@followup` として報告する。
6. **検証して統合へ渡す**: package の対象テスト／lint を実行し、最後に clean/unmerged/marker を再確認する。結果を新規 run の検証成果として記録してから evaluator に渡す。

## 検証内容と結果

- commit 親関係: pass（第2親が依存タスク target SHA と完全一致）
- 6競合の再導出: pass（remerge conflict 表示がちょうど6ファイル）
- 現在 main の差分確認: pass（1 commit、docs 1ファイルのみ）
- current-main merge 予行: pass（競合なし）
- worktree/index の未解決競合確認: pass（0件）
- conflict marker 確認: pass（0件）
- ソース変更禁止: pass（指定 worktree は調査前後とも clean）

## 未解決事項・範囲外で見つけた点

- `origin/main=3756b4f` とローカル `main=45bb579` に1 commitのずれがある。次の run はどちらを target としたかを SHA で記録する必要がある。
- ローカル main の追加は docs のみで予行上は無競合だが、実際の統合操作は agent-flow の base-sync 担当範囲であり、本タスクでは実行していない。
- ソース実装・テスト実行は本タスクの範囲外。Git 構造と競合状態の read-only 検証のみを実施した。

@followup 新規 run の base-sync receipt に target SHA（`3756b4f` または `45bb579`）を必須記録し、`main` という可変名だけで成功判定しない。

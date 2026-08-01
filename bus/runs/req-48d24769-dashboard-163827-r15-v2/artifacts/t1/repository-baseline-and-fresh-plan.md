# dashboard-163827 基準点と再実行計画

## TL;DR

`req-48d24769-dashboard-163827-r15-v2` は旧 run の結果を継承しない。現 worktree の `HEAD=ea0c808c6` は最新 `main=9c196643e` を第2親に持ち、`tools/agent-dashboard` の内容は main と同一、作業ツリーも clean だった。したがって再度の統合作業は不要で、現 HEAD から調査、契約確認、機能検証、UX レビューをすべて新規実行する。

旧 run の成果は経緯の把握にだけ使う。旧ノードの `done`、テスト結果、レビュー結果を新 run の合格証拠にはしない。終了コードが 0 でも terminal の `ok:false` があれば、そのノードは失敗として修正と再検証へ戻す。

却下した案: `r11-v2` の done ノードを再利用する案。最新 main が進んでおり、`r11-v1` では `{"ok": false}` を返した work が `status: done` になった記録もあるため採用しない。

読者: t2 以降の worker、verify、judge、synthesize の担当者。

## 完了条件と範囲

このタスクの完了条件は、Git の基準点、現 run のメタデータ、旧 run の誤完了記録を確認し、継承なしの計画と合否基準を固定すること。ソース変更、旧 run の結果の採用、agent-project のフック実装、UI からの状態書換は対象外とした。

採用した前提は二つある。最新 main は依存タスクが統合したローカル参照 `9c196643e1a8e7a3efd3e47cc36f535f782d438a` とする。「継承しない」は旧成果を調査資料として読むことまでは禁じず、旧ノードの状態や出力を現 run の完了判定へ流用しない、という意味にした。

## 確認結果

### Git

- `59ccf49e71d41fe35533ae718f010649863bd6a3` は `main` の祖先で、merge-base も `59ccf49e`。この commit は r11-v1 の base-sync で、第2親は当時の main `3756b4f`。
- `59ccf49e..main` はリポジトリ全体で 55 files、2,209 insertions、267 deletions。`tools/agent-dashboard` では 26 files、1,510 insertions、212 deletionsで、対象機能の実装とその後の main 更新を含む。
- 現 HEAD `ea0c808c66ba9f512848ffca5f160cda9a31fe77` の親は `ed4697817…` と最新 main `9c196643e…`。base-sync の記録も target `9c196643e…`、競合ファイル 0 件。
- `git diff main..HEAD -- tools/agent-dashboard` は空。現ブランチの対象ディレクトリは最新 main と同一である。
- `git status --porcelain`、`git ls-files -u`、`git diff --check main..HEAD -- tools/agent-dashboard` はすべて出力なし。競合マーカーも 0 件。

### run 記録

- 現 run は 2026-08-01T11:28:19Z 作成、status は確認時点で `running`、graph iteration は 0。`inherited/` は存在せず、結果ファイルは `base-sync.json` だけだった。
- r11-v1 は `旧run継承とok:false誤完了の修正を適用した新世代へ切替` を理由に cancelled。`base-sync-fix-11` は本文末尾で `{"ok": false}` と未完了理由を返したのに、result の status は `done` だった。
- r11-v2 は status `done` だが、途中の contractgate と uxgate は `ok:false` で failed。その後の修正とレビューも含め、現 run では一件も合格証拠として引き継がない。

## 新規実行計画

1. **統合状態の再確認**: t2 は再 merge や rebase を行わず、現 HEAD が最新 main を含むこと、対象差分がないこと、未解決競合と不要な Markdown 末尾空白差分がないことを新しい terminal で確認する。いずれかが崩れていれば失敗とする。
2. **データ経路と導線の再調査**: t3 と t4 は現 HEAD から `regression_cmd`、`intake_cmd`、codd-gate、needs-diagnosis の読み取り経路と README の設定編集、sibling CLI 導線を調べ直す。公式契約は needs、inbox、commands に限定する。
3. **現行実装との突合**: t5 で調査結果を短い判断書にまとめ、t6 から t9 は不足が見つかった場合だけ `tools/agent-dashboard` 内へ最小修正とテストを加える。すでに満たす項目も旧結果で済ませず、現 HEAD の根拠と新規検証結果を残して no-op 完了とする。
4. **独立した再検証**: regress と ux を新規プロセスで実行する。judge は両方の今回結果だけを読み、失敗があれば fix へ戻す。fix 後は gate を最初から再実行する。
5. **完了判定**: synth は今回の成果だけを統合する。最後に `echo "done"` を実行し、終了コード 0 かつ terminal `ok:true` の場合だけ done とする。

## 検証基準

### リポジトリ

- `git merge-base --is-ancestor main HEAD` が exit 0。
- `git diff --exit-code main..HEAD -- tools/agent-dashboard` が exit 0。
- `git status --porcelain=v1 --untracked-files=all` と `git ls-files -u` が空。
- `git diff --check main..HEAD -- tools/agent-dashboard` が exit 0。競合マーカー検索が 0 件。

### 機能と契約

- `regression_cmd` と `intake_cmd` の設定状態、codd-gate への結線状態、未結線時の README 準拠導線を画面で区別できる。
- 設定済みだが未結線のケースでも設定例と置換警告を表示し、既存コマンドを sibling CLI で自動置換する案内は出さない。
- needs の codd-gate と needs-diagnosis で `failureSummary`、`why`、`detail`、`failureContext.command` が読め、UI は done 状態や設定を書き換えない。
- 新しい terminal で次を実行する。

```sh
node tools/agent-dashboard/test/consistency-gate.test.js
node tools/agent-dashboard/test/consistency-gate-ui.test.js
node tools/agent-dashboard/test/needs-gate-integration.test.js
node tools/agent-dashboard/test/needs-diagnosis.test.js
node tools/agent-dashboard/test/overview-ui.test.js
npm --prefix tools/agent-dashboard test
npm --prefix tools/agent-dashboard run lint
```

各コマンドは exit 0 が必要。さらに terminal 結果が `ok:false` なら、終了コードにかかわらず失敗と記録する。失敗を既存不具合と判断しても gate は通さず、範囲外なら `@followup` として切り出す。

### UX と最終 gate

- agent-reviewer で、概要の情報階層、未結線時の対処導線、codd-gate と needs-diagnosis の可読性、状態書換を誘発しないことを新規レビューし、未解決指摘を 0 件にする。
- fix が発生した場合、対象テストだけで済ませず上記の全コマンドと UX レビューを再実行する。
- `echo "done"` は上記がすべて合格した後だけ実行する。exit 0 単独では完了にしない。

## 未解決事項

現時点でソース上の未解決事項は確定していない。main と HEAD の対象ツリーが同一でも、最新 main 上の機能が正しいことまでは Git 差分だけでは証明できないため、後続 gate の新規実行は省略しない。

`r15-v2` の meta にある legacy verification command は `echo "done"` だけで、機能回帰を検出できない。上記の regress、UX、gate を先行条件にすることで補う。

{"constraints":["旧 run の done・test・review 結果を現 run の合格証拠へ継承しない","terminal ok:false は終了コードにかかわらず失敗として記録する","tools/agent-dashboard 外を変更せず、公式契約 needs/inbox/commands 以外へ書き込まない","echo done は新規の回帰検証とUXレビューが合格した後だけ実行する"]}

# t5〜t9 dashboard 契約検証

verify=fail

## 独立検証結果

- 現 HEAD `86699a1e5` は t7 (`2a0b2eb05`) と t8 (`86699a1e5`) を含むが、t9 (`98cb4584a`) は祖先ではない（`git merge-base --is-ancestor ... HEAD` = exit 1）。
- `main...HEAD` の dashboard 差分は 3 ファイル、32 additions / 17 deletions。すべて `tools/agent-dashboard` 配下で、t7 の renderer/UI test と t8 の needs diagnosis test に一致する。t5 は変更なし。
- t9 の差分は `test/overview-ui.test.js` 1 ファイル、9 additions / 2 deletionsで、現 HEAD へ競合なく適用可能だが未統合。
- dashboard 全 `npm test` は PASS。対象4ファイルの構文検査・ESLint、`git diff --check` も PASS。worktree の未コミット差分はない。

## 公式契約との照合

- 表示データは既存の `readProject()` 経由で、公式 `needs/*.md`、`commands/*.err`、`commands/processed/*.json`、`regression_cmd` / `intake_cmd` 設定を読む。今回の production 差分は renderer の表示条件だけで、新しい読取元や書込み元は追加していない。
- UI 操作は既存の `needs` feedback、`inbox/*.json`、`commands/*.json` 投函口を維持している。今回の差分に task/backlog status や `done` を直接書く処理、新しい IPC、外部 CLI 実行はない。
- README の正規設定例と画面のコマンド文字列は一致する。

## issues

1. `tools/agent-dashboard/src/renderer/renderer.js:1232-1252` は有効化導線を `regressionConfigured` / `intakeConfigured` の欠落で判定している。そのため両キーに別コマンドが設定され、`regressionWired=false` / `intakeWired=false` の未結線状態では、設定値と「未結線」は表示する一方、README 準拠の設定編集・sibling CLI 導線を全て隠す。元要求の「未結線時は README と同じ有効化導線を示す」を満たさない。`regressionWired` / `intakeWired` を基準に未結線キーの導線を出し、既存値を自動変更せず、置換で既存処理が失われる旨を明示すること。`test/consistency-gate-ui.test.js:136-153` と t9 の `overview-ui.test.js` にある「設定済みなら導線を出さない」期待値も要求に合わせて修正すること。
2. t9 コミット `98cb4584aecb52904f5ca956eac5b5e91f7cc939` が作業ブランチの現 HEAD に統合されていない。`tools/agent-dashboard/test/overview-ui.test.js` の設定例・未結線導線に関する9行の回帰検証が納品差分に存在しない。issue 1 の期待値を正してから、このテスト差分を現ブランチへ統合すること。

{"ok": false, "issues": ["renderer.js:1232-1252 が設定済み・codd-gate 未結線時の README 準拠有効化導線を隠している。wired フラグ基準で導線を出し、既存コマンド置換の警告を表示すること。", "t9 commit 98cb4584aecb52904f5ca956eac5b5e91f7cc939 が現 HEAD に未統合。要求に沿う期待値へ直した overview-ui 回帰テストを統合すること。"]}

# final-synth-r15 最終報告

## サマリー

競合解決済みの現行 HEAD `f2042ebfe458674b75b5afcde124e98d8ce40f3a` を `main` (`9c196643e1a8e7a3efd3e47cc36f535f782d438a`) と比較し、`tools/agent-dashboard` の現行成果を統合した。古い run や解消済み gate fail は現状として扱っていない。

現行差分は 6 ファイル、226 additions / 12 deletions。

| ファイル | 現行成果 |
|---|---|
| `CONSISTENCY-GATE-DESIGN.md` | 現在の結線状態と失敗時点の記録を分離する設計、表示契約、done 不変条件、公式契約境界、検収観点を正典化 |
| `README.md` | `regression_cmd` / `intake_cmd` の正式値と、リポジトリルートから sibling CLI を使う有効化導線を明記 |
| `src/renderer/renderer.js` | README と同じ `python3 tools/agent-project/codd_gate_regression.py --config <状態 clone>/agent-project.yaml` を表示 |
| `test/consistency-gate-ui.test.js` | README と画面の設定例・CLI 引数を契約として同期確認 |
| `test/needs-gate-integration.test.js` | 公式 `consistencyGate` と `failure-*` の読取、3 状態表示、codd-gate 回帰要約、blocked 維持、無書込みを統合確認 |
| `src/features/cowork/main/cowork.js` | 競合解決時に落ちていた final-gate の未使用変数削除を復元し、現行 HEAD の lint を通過 |

本機能の表示本体は基準版に既に存在する。今回の production 差分は sibling CLI 導線の正規化と、競合で欠落した lint 修正だけであり、表示契約の追加保証は設計書と回帰テストで行った。

### 画面で判断できる内容

- 概要の「一貫性ゲート」で、全体を `結線済み` / `一部結線` / `未結線` の三値で表示し、`regression_cmd` と `intake_cmd` ごとに設定有無、codd-gate 結線状態、現在値を示す。
- 未結線時は README と同じ YAML 値を示す。`regression_cmd` は sibling CLI または設定編集、`intake_cmd` は設定編集へ案内する。画面操作は設定ファイルを `openPath` で開くところまで。
- needs の既存 `need-diag` は、要約、対処、分類、対象、コマンド、作業場所、終了コードを保持する。記録コマンドと `failure-phase` が裏付ける場合だけ `need-gate` を追加し、codd-gate の regression とタスク自身の verify を区別する。
- 現在の結線状態を過去の失敗原因へ遡及適用しない。別コマンド、根拠不足、旧票は codd-gate 由来と断定しない。

### UX レビュー結果

採用結果は pass。現在の全体状態を概要、個別の失敗事実を要対応へ置き、要対応では失敗要約と対処を先、詳細判断材料を後にした。同じ「一貫性ゲート」語彙と warning badge を両画面で使い、未結線でも現在値を隠さない。設定済みと codd-gate 結線済みを分けるため、別用途の汎用フックを有効と誤認しない。自動設定・自動実行・楽観更新は採用しなかった。

## 検証内容と結果

依存成果 `final-gate-r15/report.md` は全項目 pass だったが、記録 SHA `016a4bde9...` は現行 HEAD と別系列だった。このため pass を無条件に継承せず、現行 worktree で再検証した。再検証で競合解決時に復活した未使用変数を 1 行削除し、以下をすべて exit 0 にした。

| 確認 | コマンド / 根拠 | 結果 |
|---|---|---|
| 関連 UI | `node test/consistency-gate-ui.test.js` | pass |
| needs 統合 | `node test/needs-gate-integration.test.js` | 10 passed |
| lint 修正の回帰 | `node test/cowork.test.js` | 41 passed |
| 全体テスト | `npm test` | pass、fail 0 |
| 全体 lint | `npm run lint` | pass |
| 差分形式 | `git diff --check main...HEAD` と未コミット差分の `git diff --check` | pass |
| 競合マーカー | `tools/agent-dashboard` 全体を検索 | 該当なし |
| スコープ | `main...HEAD` と未コミット差分を確認 | `tools/agent-dashboard` の上記 6 ファイルだけ |
| final-gate-r15 | 依存 report の関連テスト、全体 test、lint、差分、競合、スコープ、done 不変条件 | 全項目 pass |

公式契約外への書込みがないことは、production 差分に設定・needs・status の書換処理がないことと、統合テストで次を確認した。

- renderer は `readProject().consistencyGate` の公式 payload だけを読み、描画前後で payload を変更しない。
- 有効化ボタンは `openPath(configFile)` だけを呼び、未設定値を楽観更新しない。
- `parseNeeds()` が読む公式 `failure-*` を表示に使い、失敗票は `kind=blocked`、`decided=false` のまま。
- dashboard から approve、complete、設定保存、codd-gate 実行、backlog 直接投入の経路を追加していない。

## 採用した前提・未解決事項・範囲外

- 前提: 「現行成果」は指定 worktree の競合解決済み HEAD とその task 内差分を指し、依存 report の古い SHA より優先した。
- 前提: final synth は設計・実装・README・テスト・UX 判断・final gate を一つの自己完結した報告へ統合し、検証で見つかった同一スコープの lint 欠落は現行成果へ復元するところまでを完了条件とした。
- 前提: done 不変条件は、dashboard が設定・needs・タスク状態・done を直接変更せず、失敗を blocked のまま表示することとした。
- 未解決: agent-project が明示 `--config` で使う実効パスと dashboard の自動探索候補が一致するかは現行の公式 status 契約では分からない。画面は断定せず候補として表示する。
- 範囲外: codd-gate の実在・バージョン互換・実行成功の監視、agent-project 本体のフック追加、intake 用注入 CLI、UI からの状態書換は実施していない。
- 範囲外で新たに見つかった問題はない。

terminal ok:true

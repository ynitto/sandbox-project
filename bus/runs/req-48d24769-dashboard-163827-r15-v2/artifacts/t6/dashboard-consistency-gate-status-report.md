# t6 成果報告

## 成果／サマリー

コード変更は不要と判断した。現 HEAD の `tools/agent-dashboard` には、概要タブの単一「一貫性ゲート」セクションが既にあり、次を満たしている。

- `regression_cmd` / `intake_cmd` を各1行で表示し、それぞれ `設定: あり` / `設定: なし` を示す。
- 設定有無とは別に `結線済み` / `未結線` を示し、別コマンドが設定済みの場合も現在値を隠さない。
- 未結線のキーだけ README と同じ YAML/JSON 設定例を示す。
- 条件を満たす場合だけ `regression_cmd` 用 sibling CLI と `--dry-run` を案内し、`intake_cmd` は設定を直接編集すると示す。
- UI 操作は自動検出した設定ファイルを開くのみで、設定・タスク状態・done を書き換えない。

実装経路は `src/features/agent-project/main/project.js` の `consistencyGateStatus()` → `readProject().consistencyGate` → `src/renderer/renderer.js` の `consistencyGateHtml()` → `src/renderer/sections/overview.js` である。既存表示を重複実装しないため、worktree 内のファイルは変更していない。

## 検証内容と結果

- `test/consistency-gate.test.js`: 15件成功。両方設定、片方設定、未設定、空値、別コマンド、YAML scalar、壊れた JSON を確認。
- `test/consistency-gate-ui.test.js`: 成功。状態バッジ、設定有無、現在値、未結線時の設定例・編集／CLI 導線を確認。
- `test/overview-ui.test.js`: 全件成功。概要への配置と既存表示との統合を確認。
- 対象6ファイルの ESLint: 問題なし。
- `git diff --check -- tools/agent-dashboard`: 成功。
- `git status --short`: 変更なし。

この worktree には `node_modules` が無かったため、依存タスク t5 の同一コミット worktree に既にある `node_modules` を `NODE_PATH` で読み取り専用参照した。最初の実行はその不足により `yaml` / `@eslint/js` を解決できず停止したが、上記の再実行は全て成功した。

## 採用した前提・未解決事項・範囲外

- 「設定済みか未設定か」はキーの非空値を指し、「一貫性ゲートに結線済みか」は codd-gate の正規コマンド形を指す。誤認を避けるため画面では両者を分ける。
- タスクの「単一の状態表示コンポーネント」は、概要の既存 `consistencyGateHtml()` を指すと解釈した。
- agent-project が明示 `--config` で使う実効設定は現契約から判定できず、画面も自動探索候補である旨を表示する。
- needs 表示、設定変更操作、agent-project 本体は範囲外として変更していない。
- 範囲外で新たな問題は見つけていない。


# t4 成果報告

## 成果／サマリー

- `tools/agent-dashboard/src/features/agent-project/main/project.js` の
  `consistencyGateStatus()` だけを変更した。
- 既存の `readProject()` スナップショット内 `consistencyGate` に
  `regressionConfigured` / `intakeConfigured` を追加した。
- renderer は既存の `dashboard:project` IPC → preload `readProject()` 経路で、
  各コマンドの設定有無、`regressionWired` / `intakeWired` / `wired`、
  表示用コマンド値を同じ読取専用スナップショットから参照できる。
- 状態更新 API、agent-project 本体、done 不変条件には触れていない。

## 検証内容と結果

- `node test/consistency-gate.test.js`: PASS（6 tests）
  - 両キー設定あり
  - 別コマンド設定ありだが codd-gate 未結線
  - キー未設定
  の各ケースで明示フラグを確認した。
- `node test/consistency-gate-ui.test.js`: PASS
- `node test/needs-gate-integration.test.js`: PASS（8 tests）
- `npm test`: PASS（終了コード 0、agent-dashboard 全テスト）
- `git diff --check`: PASS

## 採用した前提・未解決事項・範囲外

- 「設定あり」は、ワークスペース配下の既存設定契約から読んだ値が trim 後に非空であること、と解釈した。
  空文字・空白だけの値は未設定とする。
- 「一貫性ゲート判断材料」は既存の個別結線フラグ、全結線フラグ、コマンド値を維持し、
  設定有無だけを明示フラグとして補うこと、と解釈した。
- グローバル設定を対象 workspace の設定として扱わない既存契約、および
  codd-gate の語順判定契約は変更していない。
- 未解決事項なし。範囲外の agent-project フック実装、設定書換え、done 判定は変更していない。

@followup renderer 側で「設定済み」と「codd-gate 結線済み」を明示的に分ける追加表示が必要になった場合は、今回追加した `*Configured` と既存の `*Wired` を組み合わせて使用する。

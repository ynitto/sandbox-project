# t3 調査・照合結果

## 成果／サマリー

- 設定値の供給元は agent-project の公式汎用フック `regression_cmd` / `intake_cmd`。dashboard は `readToolConfig()` で実効設定を読み、`consistencyGateStatus()` で codd-gate 結線を判定し、`readProject().consistencyGate` として表示層へ渡す。
- 設定探索は agent-project 本体と同じく、workspace 直下、workspace の `.agents`、legacy `.agent`、user home の順。プロジェクト固有設定が無ければ user home の実効設定が供給元になる。
- 公式境界は読み取りと案内まで。dashboard から設定、needs、タスク状態、done を書き換えず、新規 IPC や agent-project フックも追加しない。
- 未結線時の正準手順は `.agents/agent-project.yaml` への 2 行の手編集。`regression_cmd` だけは `tools/agent-project/` の sibling CLI `python3 codd_gate_regression.py --config <existing-config>` でも冪等注入できる。`intake_cmd` の注入 CLI は無く、直接編集する。
- 現在のブランチ上の `tools/agent-dashboard` 実装・README・テストは上記契約と一致していたため、このタスクでは追加のソース変更を行っていない。

## 検証

- `tools/agent-project/README.md`、`agent_project/configfile.py`、`agent_project/needs.py`、`agent_project/mr.py`、`codd_gate_wiring.py`、`codd_gate_regression.py` と dashboard 実装／README を突合: 一致。
- `cd tools/agent-dashboard && npm test`: 全件成功（失敗 0）。一貫性ゲートの設定供給、UI 導線、needs 連携を含む。
- `git diff --check`: 成功。
- `git status --short`: clean。

## 前提・未解決・範囲外

- 前提: このタスクの完了条件を「現在の累積ブランチ実装が公式契約境界と README の有効化手順に一致することの確定」と解釈した。既存実装が条件を満たしているため、重複変更は不要と判断した。
- dashboard は設定文字列のみを判定し、codd-gate の実在、バージョン／schema 互換、実行成功は保証しない。sibling CLI は既存 YAML 設定を対象に `regression_cmd` のみを扱う。
- 未解決事項なし。
- 範囲外で見つけた事項: 依存タスク報告どおり main との rebase／競合解消は agent-flow 側の責務。本タスクでは git 履歴操作を行っていない。

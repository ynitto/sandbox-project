# t6 プロジェクト情報供給結合点

## 成果／サマリー

- 指定ブランチの `tools/agent-dashboard/src/features/agent-project/main/project.js` には、公式 `agent-project` 設定を `readToolConfig('agent-project', [workspace, ...agentDirCandidates(workspace)])` で読み、`readProject().consistencyGate` として UI へ供給する結合点が既に存在した。
- 供給モデルは `regressionConfigured` / `intakeConfigured`（非空設定の有無）、`regressionWired` / `intakeWired`（公式 codd-gate コマンドへの結線）、`wired`、表示用コマンド、設定ファイルパス、設定読込エラーを含む。汎用フックに別コマンドが設定された場合を「未設定」と誤認しない。
- 結線判定の正規表現は `tools/agent-project/codd_gate_wiring.py` の公式判定と一致する。コマンドは実行せず、設定・needs・commands・done 状態への書込み経路も追加していない。
- 未設定／未結線時の UI は、この供給モデルを使って README と同じ設定編集行と、`regression_cmd` 用 sibling CLI `codd_gate_regression.py` の案内を返す既存実装に接続済みだった。
- 現在の累積ブランチが完了条件を満たしていたため、重複するソース変更は行っていない。

## 検証内容と結果

- `node test/consistency-gate.test.js`: 13 件成功。両方／片方／未設定、別コマンド、空値、YAML block scalar、local/global 優先順、不正 JSON を確認。
- `npm test`: 全件成功（exit 0）。供給モデル、概要 UI、有効化導線、needs 連携を含む dashboard 全体の回帰なし。
- `git diff --check`: 成功。
- `git status --short`: 出力なし（作業ツリー clean）。
- `project.js` の判定式と `tools/agent-project/codd_gate_wiring.py` の `_REGRESSION_WIRED_RE` / `_INTAKE_WIRED_RE` を目視突合し、一致を確認。

## 採用した前提・未解決事項・範囲外

- 完了条件は「公式契約から設定有無と codd-gate 結線状態を読み取り専用のプロジェクト情報として UI に供給し、既存 UI が README と同じ有効化導線を返せること」と解釈した。
- 「設定有無」と「codd-gate 結線済み」は別の事実として扱う。`regression_cmd` / `intake_cmd` は汎用フックなので、非空だけでは一貫性ゲート有効と判定しない。
- dashboard は設定文字列だけを判定し、codd-gate の実在、バージョン／schema 互換、実行成功までは保証しない。
- 未解決事項なし。
- agent-project 本体、公式 needs/inbox/commands への書込み、done 状態変更には触れていない。範囲外で追加対応が必要な問題も見つからなかった。

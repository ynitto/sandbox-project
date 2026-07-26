# 統合結果

- gate の blocking 2 件を現物で再現し、依存報告の「変更不要」との矛盾は gate 側を採用して解消した。
- `consistencyGate` は agent-project 本体と同じ実効設定を採用するよう修正した。プロジェクト root 解決だけは従来どおり global `root` を無視し、ゲート設定の global fallback とは分離した。
- global fallback と workspace 優先順位、概要への実埋込み、`bindConsistencyGate` から `api.openPath` へのクリック配線を退行テストで固定した。
- README を実効設定の探索順に合わせ、画面の有効化導線（設定編集、regression 用 sibling CLI、intake の直接編集）との意味のずれを解消した。
- 競合インデックス、コンフリクトマーカー、ゲート関数の重複実装は残っていない。変更は `tools/agent-dashboard/` 配下のみ。

検証: 対象テスト成功（consistency-gate 7/7、overview UI、consistency-gate UI、needs gate 8/8）、`npm test` 全件成功、`git diff --check` 成功。

{"constraints":["プロジェクト root 解決では複数プロジェクト表示を壊す global root を採用しない一方、regression_cmd/intake_cmd のゲート表示は agent-project 本体と同じ実効設定（workspace 優先、未設定時は共通ホーム fallback）を採用する。","dashboard のゲート操作は読み取りと設定ファイルを開く導線に限定し、設定書換・コマンド実行・done 状態変更を行わない。"]}

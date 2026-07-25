# synth2 統合結果

verify=pass

- t8/t9/t10 の変更は競合なく統合済み。変更範囲は `tools/agent-project` 配下7ファイルのみ。
- 自動配線は `_hook_provider` を唯一の汎用境界とし、明示設定は `_hook_import`、未指定は `_hook_scan_siblings` のみを使う。
- doctor は `wiring.detect -> detect_wiring` と `wiring.findings -> doctor_findings` を同じ境界で解決し、provider の判定値・findings を透過、不在時は no-op。
- model/intake は provider 固有名を持たず、汎用 JSON レコードの検証と `id` 冪等を維持。
- regression gate は `run_verify` で失敗を `blocked`、成功を `done` へ遷移させる。
- 旧識別子・`agent_project` 内の `codd_gate` import は0件。`git diff --check` 成功。
- 全テスト 969/969、受入3ケース 3/3 成功。
- 元要求の個別テストパス `tests/test_agent_project.py` は現行ツリーに存在しないため、実在する分割先 `tests.test_backlog` / `tests.test_autonomy` で同一ケースを実行した。

追加変更なし。gate2 の結論と依存成果の件数・契約に矛盾なし。

{"ok": true, "issues": []}

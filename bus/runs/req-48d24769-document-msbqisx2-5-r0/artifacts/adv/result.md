# verify=fail

## Issues

1. `tools/agent-project/agent_project/_head.py:143-146` の PROMPT 正規表現は、引用符なしの複数語 `raw_prompt=` について最初の空白までしか置換しない。実測では `raw_prompt=RAW_PROMPT_FIXTURE: publish the confidential roadmap verbatim` の後半 `publish the confidential roadmap verbatim` が残り、`assert_share_safe` も通過した。したがって `brief.py:68` と `decisions.py:56` の両保存経路、および state git の再検査で生プロンプト漏出を阻止できず、README.md:343-372 の設計（生プロンプトを共有禁止、置換後再検査）とも不一致。PROMPT の値境界を明確にして本文全体を置換し、複数語・改行・引用あり/なしを通す契約テストを追加すること。
2. `tools/agent-project/tests/test_privacy.py:12-19` は sensitive 値の完全一致だけを `assertNotIn` しているため、上記の部分漏出を検出しない。token も `token=...` というキー付き入力しかなく、`TOKEN_FIXTURE_4f8a2d6c_NEVER_REAL` 単体および識別用部分文字列 `4f8a2d6c_NEVER_REAL` は `_share_categories` で未検出だった。要求どおり token の識別可能な部分文字列が brief、decisions、state 共有候補の全ファイルに残らないことを assert し、token の単体・埋め込み形も fixture から各入口へ流すこと。

## Confirmed

- 差分は `tools/agent-project/` 配下の 7 ファイルのみ。`git diff --check main...HEAD` 成功、作業ツリー clean。
- privacy テスト 3 件成功。全スイート 1161 件成功（385.765 秒）。
- `.github/workflows/ci.yml:31-32,56-61` が agent-project tests を `unittest discover` で実行するため `test_privacy.py` は CI に収集される。
- `redact_for_share` を恒等変換へ差し替える負例で privacy テストが失敗したため、現在 fixture が捉える違反は CI の非ゼロ終了へ伝播する。
- state fixture 混入時、ローカル HEAD と bare remote の main はともに不変で、commit/push 前停止を確認。

{"ok": false, "issues": ["_head.py:143-146 の PROMPT 正規表現が引用符なし複数語の先頭だけを置換し、生プロンプト後半が brief/decisions/state 検査を通過する", "test_privacy.py:12-19 が完全一致のみを検査し、token の部分一致・単体/埋め込み形と生プロンプトの部分漏出を検出しない"]}

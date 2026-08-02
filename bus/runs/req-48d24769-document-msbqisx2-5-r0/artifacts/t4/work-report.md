# t4 成果報告

## 成果／サマリー

- `agent_project/_head.py` に共有前 redaction の単一正本を追加した。token、POSIX/Windows ホームパス、生プロンプト、生資格情報をカテゴリ別 marker へ置換し、同じ検査器で置換後を再検査する。検査器異常は fail-closed とし、例外には相対パスとカテゴリだけを含める。
- `append_brief_item` は正規化直後、重複判定・brief 追記・journal 追記より前に redaction を一度適用する。journal へ渡るのも置換済み本文だけである。
- `append_decision` は完成した DR block 全体へ、ファイル追記前に redaction を一度適用する。
- `DirectStateGit` は同期除外規則を再利用して共有候補ツリーを検査する。初回 commit、通常 worktree commit、transaction の add 前、全 push の直前を fail-closed にした。push 前は outgoing commit の各 snapshot を検査するため、後で削除された未 push 中間 commit の blob も拒否する。
- 変更は `tools/agent-project/agent_project/{_head.py,brief.py,decisions.py,stategit.py}` の4ファイルのみ。commit/push/branch 操作は未実施。

## 検証内容と結果

- `python -m py_compile`（変更4ファイル）: 成功。
- `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests -p 'test_delivery.py'`: 成功。
- `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests -p 'test_state_git.py'`: 成功。
- `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests`: 成功（既存 suite 全件）。
- 架空 sentinel による最小スモーク検証: 4カテゴリの marker、POSIX/Windows home、safe 値維持、元値非残存、例外メッセージへの元値非混入を確認して成功。
- `git diff --check`: 成功。

## 採用した前提・未解決事項・範囲外

- synth の `integration-decision.md` を正本とし、「状態リポジトリ」は `DirectStateGit` の commit/push 全 egress、「共有ファイルを書き出さない」は brief/decisions の追記前および state git の commit/push 前で拒否すること、と解釈した。
- task scope の `[out_of_scope] fixture、CI` を優先した。プライバシー fixture と `test_privacy.py` の追加、および CI workflow の変更は本成果に含めていない。既存 CI は `tools/agent-project/tests` を discover するため、後続の契約テスト追加だけで CI 接続されるという synth 判断を維持した。
- 個別呼出元の重複ガード、追加依存、設計書変更、無関係なリファクタリングは行っていない。
- 未解決事項: この担当範囲単独では、全体完了条件の「privacy fixture を用いた契約テスト追加」は未達。後続の fixture/test 担当で実装する必要がある。

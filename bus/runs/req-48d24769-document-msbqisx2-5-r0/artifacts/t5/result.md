# t5 成果報告

## 成果／サマリー

- `tools/agent-project/tests/test_privacy.py` を単一契約テストモジュールとして追加した。
- t2 の `PRIVACY_FIXTURE` を実際の `append_brief_item` と `append_decision` に入力し、4カテゴリの元値が生成ファイルへ残らないこと、カテゴリ別 marker が生成されること、安全な4値が保持されることを検証する。
- 同じ fixture を実 Git リポジトリの共有候補ファイルへ混入し、baseline を push 済みの状態から `DirectStateGit.sync(force=True)` が拒否すること、local HEAD と remote ref の双方が不変であることを検証する。
- 置換後検査が残存を検出した場合は brief を一切作らず `ShareSafetyError` を送出する fail-closed 契約を検証する。
- 既存 GitHub Actions は `tools/agent-project/tests` を `unittest discover` するため、この回帰は追加の CI 設定なしで CI failure になる。

## 検証内容と結果

- `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests -p 'test_privacy.py' -v`: 成功（3 tests）。
- `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests`: 成功（終了コード 0）。
- `python -m py_compile tools/agent-project/tests/test_privacy.py`: 成功。
- `git diff --check`: 成功。
- `git status --short`: 新規 `tools/agent-project/tests/test_privacy.py` のみ。許可範囲外のリポジトリ変更なし。

## 採用した前提・未解決事項・範囲外

- 「状態リポジトリの実生成経路」は、fixture を共有候補スナップショットへ直接置き、実 Git と `DirectStateGit.sync` の commit/push 境界を通すこと、と解釈した。状態 Git は禁止値を置換せず拒否する設計なので、安全値保持と marker 生成は brief/decisions 側で検証した。
- 「redaction 残存時の fail-closed」は、置換後の共通検査器が失敗した状況を差し込み、共有ファイルが生成されないこと、と解釈した。
- fixture の生資格情報も詳細要件に含まれるため、完了条件に明記された token・ホームパス・生プロンプトに加えて同じ契約で検証した。fixture はすべて架空値であり、報告には値を転記していない。
- 本体実装、CI 設定、設計書、他機能テストは変更していない。未解決事項および範囲外で見つけた問題はない。

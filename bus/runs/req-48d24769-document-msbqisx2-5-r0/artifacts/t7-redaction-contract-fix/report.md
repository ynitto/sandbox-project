# t7 redaction contract fix

## 成果／サマリー

- `tools/agent-project` の共有前 redaction を修正した。token は単体値と単語境界のない文字列内埋め込みを検出し、生プロンプトはラベル後の行全体を除去する。
- brief は長さ制限・改行畳み込みの前に redaction し、正規化後にも共有前検査を行う。これにより生プロンプトの残片を残さず、安全な別行の値は保持する。
- privacy fixture に token 単体、token 埋め込み、生プロンプト全文と部分文字列、POSIX/Windows ホーム、URL 型の生資格情報、安全値を追加した。
- brief、decisions、状態リポジトリの commit/push 前検査を全 fixture ケースで契約化した。安全値保持の検証も継続している。
- 変更は許可範囲内の4ファイルのみ。commit、push、branch 操作は行っていない。

柱1 / C1 — ノード間で共有する state、brief、decisions から秘密・ローカルパス・生プロンプトを排除する。

## 検証内容と結果

- privacy 契約: `python3 -m unittest discover -s tools/agent-project/tests -p 'test_privacy.py' -v` — 4 tests、PASS。
- 通常 CI 相当: `AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests` — 1,162 tests、PASS（361.314 秒）。既存テスト由来の ResourceWarning はあったが失敗なし。
- 漏出変異プローブ: redaction パターンを実行時に無効化し、6 fixture × 2 経路（brief/decisions と状態リポジトリ）を実行 — 12/12 で期待どおり契約失敗を検出。
- 旧 token 境界プローブ: 旧 `\b` パターンが新しい埋め込み fixture を見逃すことを確認。
- `git diff --check` — PASS。

## 前提・未解決事項・範囲外

- 「全共有経路」は、指定された状態リポジトリの同期前検査、および本文生成を担う brief/decisions の2経路と解釈した。
- fixture の資格情報・token はすべて架空値とし、実在する機密情報は使用していない。
- 未解決事項なし。範囲外で新たに修正が必要な問題は見つからなかった。

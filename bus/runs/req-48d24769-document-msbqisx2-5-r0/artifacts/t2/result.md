切り口: 漏えい値の除去だけでなく、安全値の保持（過剰 redaction 防止）も同じ fixture で検証できる構成にした。

## 成果／サマリー

- `tools/agent-project/tests/_privacy_fixture.py` を追加した。
- 再利用可能な単一記号 `PRIVACY_FIXTURE` に、すべて架空の token、絶対ホームパス、生プロンプト、生資格情報を `sensitive` として収録した。
- task ID、status、相対パス、公開 URL を `safe` として収録し、後続の契約テストで保持を検証できるようにした。
- 変更は許可範囲内の新規 fixture 1 ファイルのみ。本体 redaction、CI、設計書は変更していない。

## 検証内容と結果

- `python -m py_compile tools/agent-project/tests/_privacy_fixture.py`: 成功。
- fixture の import、トップレベル区分、機密値 4 種、ホームパスの絶対パス性、機密値と安全値の非重複を assert する自己検査: 成功。
- `git status --short`: 対象 fixture だけが未追跡であり、許可範囲外の変更なし。

## 前提・未解決事項・範囲外の問題

- 本担当は `[scope] tools/agent-project の既存テスト fixture 用単一ファイルまたは fixture 記号` と解釈し、後続の redaction テストがこの記号を import する前提とした。
- ホームパスは CI で決定的に扱える架空の POSIX 絶対パスとした。実ユーザーのホームパスや実資格情報は含めていない。
- fixture を消費する redaction テスト、本体 redaction 実装、CI 設定は明示された担当範囲外のため未変更。CI 失敗の完了条件は、後続テストが通常の `unittest discover` 対象として追加されることで満たす必要がある。
- 範囲外で追加の問題は見つけていない。

# t8 redaction adversarial verify

## 成果／サマリー

**verify=pass**

t7（commit `031b4d2158e89bd09bbadb74ac14077ab458233e`）を独立に再検証した。token、ホームパス、生プロンプト、生資格情報の各検出カテゴリを実行時に1つずつ無効化して共有出力へ fixture の禁止値を残す変異を作ったところ、4/4カテゴリで CI 相当の privacy テストが終了コード `1` になった。状態リポジトリ、brief、decisions の全対象経路で検出を確認した。無変異では privacy 契約と agent-project 全テストが pass した。

検出漏れ、部分漏出、要求範囲外の変更は見つからなかった。検証のためのリポジトリ変更は行っていない。

## 検証内容と結果

- 無変異 privacy 契約: `python3 -m unittest discover -s tools/agent-project/tests -p 'test_privacy.py' -v` — 4 tests、終了コード `0`、PASS。
- 漏出変異（brief + state repository）: `_SHARE_REDACTIONS` から対象カテゴリだけを実行時に除外し、同じ privacy 契約を実行。
  - token カテゴリ — 終了コード `1`。token 単体・文字列内埋め込みの brief 漏出、および両ケースの state sync 非拒否を検出。
  - home カテゴリ — 終了コード `1`。POSIX・Windows ホームパスの brief 漏出、および両ケースの state sync 非拒否を検出。
  - raw prompt カテゴリ — 終了コード `1`。brief 漏出と state sync 非拒否を検出。
  - raw credential カテゴリ — 終了コード `1`。brief 漏出と state sync 非拒否を検出。
- 漏出変異（decisions 単独）: 同じ4カテゴリを個別に無効化し、`append_decision` が生成した共有ファイルを契約 assertion に通した。token、home、raw prompt、raw credential の全4コマンドが終了コード `1`。token と home の2 fixture もそれぞれ subtest で検出。
- 部分漏出: fixture は token の埋め込み値、生プロンプトの意味を持つ部分文字列、生資格情報の秘密部分、ホームパスのユーザー領域 prefix を個別の禁止値として検査する。無変異 privacy 契約はすべて PASS、対応カテゴリ無効化時はすべて FAIL。
- CI 接続: `.github/workflows/ci.yml` の agent-project job は `python -m unittest discover -s tools/agent-project/tests` を実行するため、`test_privacy.py` は通常 CI の収集対象。上記変異の非0はその同じ unittest 契約による。
- 無変異の通常 CI 相当: `AGENT_FLOW_STUB_SLEEP_MAX=0 python3 -m unittest discover -s tools/agent-project/tests` — 1,162 tests、362.884秒、終了コード `0`、PASS。既存テスト由来の ResourceWarning はあったが失敗なし。
- 差分健全性: `git diff --check <t7^> <t7>` — PASS。検証後の `git status --short` は空。
- 変更範囲: t7 の変更は次の4ファイルのみで、すべて許可された `tools/agent-project` 配下。
  - `tools/agent-project/agent_project/_head.py`
  - `tools/agent-project/agent_project/brief.py`
  - `tools/agent-project/tests/_privacy_fixture.py`
  - `tools/agent-project/tests/test_privacy.py`

## 前提・未解決事項・範囲外

- 「CI が終了コード非0」は、リポジトリの GitHub Actions が実行する agent-project unittest discovery と同一のテスト契約が、漏出変異を含む場合に非0を返すこと、と解釈した。外部 GitHub Actions の起動は不要とした。
- 「brief/decisions の双方」は、通常テストの両ファイル assertion に加え、brief と decisions を分けた変異実行でも確認する、と解釈した。
- 変異はプロセス内の検出カテゴリ差し替えと一時ディレクトリだけで行い、作業ツリーへ禁止値や検証コードを残していない。
- fixture は架空値のみ。報告には禁止値そのものを記載していない。
- 未解決事項・範囲外で新たに見つかった問題なし。

{"ok": true, "issues": []}

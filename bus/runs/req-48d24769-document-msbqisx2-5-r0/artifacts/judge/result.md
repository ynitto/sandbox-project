# judge=fail

## 比較

| dep id | 要求適合 | 正確さ | 完成度 | 判定 |
|---|---|---|---|---|
| t3 | 二段ゲートと fail-closed 契約は要求に適合 | 設計記述は妥当 | 実装・fixture・CI 検証を担当外としており単独では未完 | 次回再実行不要 |
| t6 | 現行 `test_privacy.py` が既存 CI に収集され、違反時の非ゼロ終了が伝播する点は正しい | 3契約テスト・全1161テストの成功を再確認 | 契約テスト自体の検出漏れを評価しておらず、完了判定には不足 | 次回再実行不要 |
| adv | 未引用複数語 raw prompt と単体・埋め込み token の漏出を実例で検出 | 両反例を再現でき、指摘は正確 | 修正対象と追加すべき契約ケースを特定 | 最良 |

## 機械ゲート

- 契約テスト: 3件成功。
- 既存テスト / CI 相当: `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests` は1161件成功、終了コード0。
- adversarial raw prompt: `[REDACTED:PROMPT] publish the confidential roadmap verbatim` が残り、再検査を通過。反例コマンドは終了コード1。
- adversarial token: token 単体と識別断片の埋め込みが `_share_categories` で未検出。反例コマンドは終了コード1。
- 差分健全性: `git diff --check main...HEAD` 成功。変更は `tools/agent-project/` 配下のみ。

## 判定と戻し先

不合格。成功している現行契約テストと CI 配線だけでは、生プロンプトと token の部分漏出を検出できない。さらに state-git の remote 受信経路には共有前検査がなく、禁止値を含む snapshot を materialize できる。

最小再実行ノードは `t4`（redaction と state 受信境界の修正）→ `t5`（複数語・複数行・引用有無、token 単体・埋め込み・識別断片、remote fast-forward 受信の契約 fixture）→ `adv`。設計のみの t3 と CI 配線確認のみの t6 は再実行しない。

{"winner":"adv","constraints":["変更可能範囲は tools/agent-project 配下のみ。CI 定義 .github/workflows/ci.yml は読取確認のみ可能だが、既存 unittest discovery が契約テストを収集するため変更不要"]}

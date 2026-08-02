# agent-project redaction 調査結果

## (a) 成果／サマリー

### 前提

- このノードの担当は調査と変更境界の確定であり、明示された `out_of_scope` に従ってコード変更・テスト追加は行わない。元要求の完了条件は後続の実装タスクが満たす。
- 調査対象は指定 worktree の `tools/agent-project`。CI への接続確認に限り `.github/workflows/ci.yml` を読み取り、他ツール配下は調査していない。
- `.codegraph/` はリポジトリルートに存在しなかったため、CodeGraph は使用せず `rg` とソース読解へ切り替えた。

### 結論

現時点で redaction の実装・fixture・契約テストは存在しない。存在するのは README の契約だけで、置換記号は次の4つに確定済み。

- `[REDACTED:TOKEN]`
- `[REDACTED:HOME]`
- `[REDACTED:PROMPT]`
- `[REDACTED:CREDENTIAL]`

変更対象の単一の共有前検査境界は、**同じ redaction／検査プリミティブを唯一の正本とし、共有本文の追記直前と state snapshot の commit/push 直前から必ず呼ぶ境界**とする。断片のロード順では `decisions` が `brief` より先なので、共通プリミティブは両者より前にロードされる `agent_project/_head.py`（または同位置に挿入する専用 fragment）に置く必要がある。個別入口ごとに regex を複製してはならない。

具体的な適用点は以下。

1. `agent_project/brief.py:64` の `append_brief_item`：`_norm_brief_item` 後、重複判定・ファイル追記・journal 出力より前。ここを通すと feedback、revise、forge/gitlab reject、cohort、node constraint を一括で覆う。journal は `brief.py:87` で本文先頭80文字も出すため、必ず redaction 後の `body` を使う。
2. `agent_project/decisions.py:35` の `append_decision`：`context/action/reason/affects/learn/avoid` から block を組み立てた後、`path.open("a")` より前。全 decision 呼出元がここへ収束する。
3. state git：同期除外適用後の**共有候補スナップショット全体**を同じプリミティブで fail-closed 検査する。通常同期だけでなく CAS transaction と push-only 再送も対象にする。

`DirectStateGit.sync` 一箇所へ置くだけでは不十分である。次の迂回経路があるため、共通プリミティブを各 egress から呼ぶ必要がある。

- unborn 初回 commit：`stategit.py:316-341` `_initial_commit`
- 通常／amend commit：`stategit.py:388-438` `_worktree_commit`
- coordination の CAS commit+push：`stategit.py:177-218` `transaction`（`coordination.py:79-92` から到達）
- 既に ahead の HEAD を新規 commit なしで push：`stategit.py:960-970` `sync` の push 分岐

したがって state 側の正しい境界は「`sync()` の入口」ではなく、**commit 対象 tree と push 対象 HEAD tree を検査する共通 preflight**である。検査失敗時は commit/push を実行せず非ゼロ相当（例外または失敗値）で返し、ログにはファイル相対パスと検出区分だけを残す。

### 呼出経路

共有ファイル生成から再注入・共有までの主要経路は以下。

```text
needs feedback (needs.py:541,545) ─┬→ append_brief_item → brief/<id>.md
commands revise (commands.py:670,724) ┤
forge/gitlab reject (mr.py:486,490,810,819)
cohort/node constraint (model.py:503, mr.py:899)
                                  ├→ capture_insight (brief.py:124)
                                  │    ├→ append_brief_item
                                  │    └→ append_decision (learn 有効時)
全 decision 呼出元 ────────────────┴→ append_decision → decisions/<id>.md

brief/<id>.md ─→ brief_context (brief.py:92) ─┐
decisions/<id>.md ─→ decision_context          ├→ build_request (request.py:194)
                                                └→ agent-flow / board の全ワーカーへ注入

状態 root の変更 ─→ state_sync (loop.py:105)
  └→ DirectStateGit.sync (stategit.py:926)
      ├→ _changed_targets（除外適用）
      ├→ _initial_commit または _worktree_commit
      └→ integrate → HEAD push

coordination ─→ state_transaction ─→ DirectStateGit.transaction ─→ temp worktree commit+push
```

`brief` は書き込み後すぐ `build_request` で生プロンプトへ再注入され、`decisions` も `decision_context` 経由で再注入される。このため state git だけの検査では遅く、追記前 redaction が必須である。

### 関連テストと追加先

- brief の既存単体テスト：`tests/test_delivery.py` の `TestRunBrief`、`TestCaptureInsightAndRetireBrief`
- decision の既存単体テスト：`tests/test_decisions.py`
- state commit/push の既存単体・統合テスト：`tests/test_state_git.py`（`TestStateSyncBatching`、`TestDirectStateGit` ほか）
- 推奨される新規契約テスト：`tests/test_redaction.py` 一ファイル。実在しない sentinel のみを持つ fixture を同ファイル内または `tests/fixtures/privacy/` に置く。

最低限固定すべきケースは次の通り。

- token、POSIX home、Windows home、生プロンプト、生資格情報を `append_brief_item` と `append_decision` に流し、元 sentinel が `brief/`、`decisions/`、journal を含む共有候補ファイルに無いこと。
- 各カテゴリの置換記号が現れ、同じ入力に含めた無害な本文は残ること。
- 共有 snapshot へ禁止 sentinel を直接置き、初回 commit、通常 commit、transaction、push-only の全経路が commit/push せず失敗すること。
- エラー文字列に sentinel や該当行本文が含まれず、相対パスとカテゴリだけが含まれること。
- 読めない対象、検査器例外、置換後の再検査失敗が fail-closed になること。

## (b) 検証内容と結果

- CodeGraph 判定：`.codegraph/` なし。
- 実装検索：`tools/agent-project/agent_project` と `tests` に `REDACTED` / `redact` / `redaction` の実装・テストなし。README のみ該当。
- 呼出元検索：brief 系は `append_brief_item`／`capture_insight`、decision 系は `append_decision` に収束することを確認。
- state egress 読解：`_initial_commit`、`_worktree_commit`、`transaction`、push-only の4経路を確認。
- CI 相当コマンド：`AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests`
- 実行結果：**1158 tests、346.238秒、OK**。既存の `ResourceWarning` は出たが失敗なし。
- GitHub Actions：`.github/workflows/ci.yml:16-20` で `main` push／PR／手動実行、同ファイル `31-32,56-61` で agent-project matrix が `tools/agent-project/tests` を `unittest discover` する。新テストを同ディレクトリへ置けば redaction 失敗で job が失敗する。追加の CI 設定変更は不要。
- 作業 tree：変更なし。指定リポジトリには一切書き込んでいない。

## (c) 未解決事項・範囲外で見つけた問題

- **生プロンプトの識別規則が未定義**。任意の自然文だけから「生プロンプト」を安全に識別することはできない。後続実装では、prompt 由来の値を構造的にタグ付けして常に `[REDACTED:PROMPT]` にするか、明示 sentinel／フィールド規約を契約として定義する必要がある。単なる fixture 固有文字列 regex では実運用の安全網にならない。
- 「token」は認証 token に限定し、コストの `tokens=...` や分散制御の `claim_token` 名まで一律置換しない識別規則が必要。
- README は「過去の漏出を検出した場合も同期を止める」とするが、現行 tree の検査だけで到達可能な過去 commit 全履歴まで走査するかは未定義。少なくとも push 対象 commit range と現在 tree は検査対象にすべき。
- state snapshot にはテキスト以外が入り得る。バイナリ／decode 不可を一律 fail-closed にするか、許可されたバイナリ型を設けるかは契約化が必要。README の現行文言どおりなら読み取り不能は fail-closed。
- 既存テストで `ResourceWarning`（未 close file／実行中 subprocess）が観測されたが、本タスク範囲外のため修正していない。

# redaction 統合判断

## 確定事項

redaction の単一正本は `agent_project/_head.py` に置く。新規依存やクラスは追加せず、共有名前空間から次の既存経路が同じ検出・置換関数を呼ぶ。

1. `brief.py:append_brief_item`
   - `_norm_brief_item` の直後、重複判定・ファイル追記・`append_journal` より前に置換する。
   - journal には置換済み本文だけを渡す。
2. `decisions.py:append_decision`
   - DR block の完成後、ファイル追記より前に block 全体を置換する。
3. `stategit.py:DirectStateGit`
   - 自動置換はせず fail-closed とする。
   - `_initial_commit`: commit 作成前に、同期除外適用後の候補ツリー全体を検査する。
   - `_worktree_commit`: detached worktree に候補スナップショットを構築後、commit 前に全体を検査する。
   - `transaction`: `mutate(worktree)` 後、`git add` より前に全体を検査する。
   - `sync` の push-only を含む全 push: push 直前に、`origin/<branch>..HEAD` の全 outgoing commit/blob を検査する。初回 push は送信される到達可能範囲を検査する。

ツリー検査は `_changed_targets` と同じ除外規則（ドット始まりの segment、`flow-archive/`、`claims/`）を使う。Git 操作は既存 `_git_run` / `DirectStateGit._git` と harden 済み環境を再利用する。検査失敗は元値や該当行を出さず、相対パスとカテゴリだけを返す。

## fixture 契約

t2 の `PRIVACY_FIXTURE` を統合元とするが、その commit は現 HEAD の祖先ではないため未統合扱いとする。次の最小補正が必要。

- `sensitive`: 架空の token、POSIX home、生プロンプト、生資格情報に加え、架空の Windows home（例 `C:\\Users\\privacy-fixture-user`）を追加する。
- `safe`: task ID、status、相対パス、公開 URLを維持する。
- 実 token、実ホーム、実資格情報は fixture・例外・テスト出力へ入れない。
- 置換記号は README の `[REDACTED:TOKEN]`、`[REDACTED:HOME]`、`[REDACTED:PROMPT]`、`[REDACTED:CREDENTIAL]` を正本とする。

## 契約テスト対象

`tools/agent-project/tests/test_privacy.py` から fixture を import し、次を固定する。

| 対象 | 必須アサーション |
|---|---|
| `append_brief_item` | `brief/<id>.md` と journal に全 sentinel の元値が無く、対応する置換記号と safe 値が残る |
| `append_decision` | `decisions/<id>.md` に全 sentinel の元値が無く、対応する置換記号と safe 値が残る |
| 初回 sync | 禁止値を含む候補で HEAD が作られず、同期失敗になる |
| 通常 worktree sync | ローカル HEAD と remote HEAD が不変で、push されない |
| `transaction` | `False` を返し、remote HEAD が不変 |
| push-only | 禁止値追加後に削除した未 push 中間 commit を含む履歴を拒否し、remote HEAD が不変 |
| fail-closed | 読み取り不能・検査器例外・置換後再検査失敗で書込/commit/push が起きない。エラー文字列に元値が無い |

テストは caller ごとに重複させず、集約入口と全 egress を直接検証する。`capture_insight`、commands、needs は既存テストで入口への接続を担保し、新規 privacy テストでは再実装しない。

## CI 対応関係

| 検査 | コマンド | CI 接続 |
|---|---|---|
| privacy 単体 | `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests -p 'test_privacy.py'` | 実装時の短い確認 |
| agent-project 回帰 | `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests` | `.github/workflows/ci.yml` の `python (agent-project)` が PR と `main` push で同じ suite を実行 |

`test_privacy.py` を既存 discover 配下へ置けば redaction 失敗は unittest の非ゼロ終了として CI を失敗させるため、CI workflow の変更は不要であり、許可範囲外の `.github/` も触らない。

## 矛盾・欠落の解消

- t3 の二段ゲートは採用するが、「状態スナップショット」だけでは未 push 中間 commit の漏出を防げないため、outgoing commit/blob 検査を追加した。
- t1 の `_changed_targets()` 直後という案だけでは transaction と push-only を覆えないため、上記4 egress に具体化した。
- t2 fixture は内容を再利用するが、現ブランチ未統合と Windows home 欠落を解消してから契約 fixture とする。
- 現 HEAD は README の設計変更だけで、実装・fixture・契約テストは未存在。したがって総合完了条件はまだ未達であり、後続実装は上記境界とテスト対応を一括で満たす必要がある。

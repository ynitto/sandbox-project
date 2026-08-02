---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqisx3-6
kind: plan-review
---

# 実行前レビュー: document-msbqisx3-6 — dashboard に fleet/knowledge ビュー列を追加し状態リポジトリと突合可能にする

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : dashboard に fleet/knowledge ビュー列を追加し状態リポジトリと突合可能にする
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. fleet 画面に新列が追加され、status/<node>.json の値と一致する
    2. knowledge 画面で観測 provenance と適用統計が確認できる
    3. dashboard-163827 の範囲外の二次書き込みは行わない（dashboard は書き手にならない）
- why: Phase5 のユーザビリティ要件。UI で一度に判断できることが運用の前提になる。
- desc: fleet 画面に capacity bucket・鮮度(updated_iso+fresh_after_sec)・reservation・reason_code を追加、knowledge 画面に provenance・適用数・PASS/FAIL/rollback 集計・状態遷移を表示する。データ源は status/<node>.json（board はオプション）に限定する。
- after: dashboard-163827
- charter: v1
- assess: c=3 r=2 a=2
- priority: 6
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqisx3-6`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqisx3-6 --reason ...`。 -->

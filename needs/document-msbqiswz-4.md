---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqiswz-4
kind: plan-review
---

# 実行前レビュー: document-msbqiswz-4 — observation envelope（観測 sidecar）を導入し観測の idempotent 取込を実装する

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : observation envelope（観測 sidecar）を導入し観測の idempotent 取込を実装する
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. observation sidecar フォーマットが schema として追加されている
    2. 同一 observation ID を複数回取り込んでも候補集合と hit 集計が変わらない E2E fixture がある
    3. provenance（発生 node→run→receipt）を辿れるサンプルが確認できる
- why: Phase3 の基盤。観測の追跡性と重複耐性が無いと rule ライフサイクルが破綻する。
- desc: run brief/archive/decisions に共通の observation sidecar（identity, input, outcome, candidate, privacy）を定義・保存し、観測は observation ID で冪等に取り込めるようにする。git マージ順に依存しない集計結果を保証する。
- charter: v1
- assess: c=3 r=3 a=2
- priority: 4
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqiswz-4`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqiswz-4 --reason ...`。 -->

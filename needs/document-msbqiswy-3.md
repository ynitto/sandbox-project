---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqiswy-3
kind: plan-review
---

# 実行前レビュー: document-msbqiswy-3 — 割当・claim 判定に budget_summary.can_accept と鮮度判定を追加し reason_codes を出力する

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : 割当・claim 判定に budget_summary.can_accept と鮮度判定を追加し reason_codes を出力する
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. 割当処理が can_accept=false あるいは期限切れのノードへ割当を行わない fixture がある
    2. journal/ダッシュボードに reason_codes が記録される
    3. 既定で enforce=false にして差分監査が可能になっている
- why: Phase1 の割当安全化。可読な reason_codes があると監査と UI 表示が容易になる。
- desc: allocate_distributed_tasks と claim_distributed_task の適格性ロジックに node-budget-summary の can_accept と updated_iso/freshness チェックを加える。決定理由を reason_codes として journal/dashboard に出力する。
- charter: v1
- assess: c=3 r=3 a=2
- priority: 3
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqiswy-3`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqiswy-3 --reason ...`。 -->

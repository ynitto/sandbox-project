---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqiswx-2
kind: plan-review
---

# 実行前レビュー: document-msbqiswx-2 — node-budget 集約実装を agentcore に一本化する

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : node-budget 集約実装を agentcore に一本化する
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. 単一の agentcore API が存在し5つの呼び出し元がそれを利用するようになっている
    2. 同一 fixture に対して旧実装と同等の出力を返す突合テストが追加され、合格している
    3. 重複実装のコードブロックが削除または呼び出しに置換されている
- why: C7 回復の核心。判断ロジックを一箇所に置けば将来の維持コストと不整合を無くせる。
- desc: node-budget の読取・推定・state 計算を agentcore へ移し、agent-amigos/agent-flow/agent-project/agent-loop/ダッシュボードに対し共通 API を提供して既存の 5 箇所の重複実装を置換する。
- charter: v1
- assess: c=3 r=2 a=2
- priority: 1
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqiswx-2`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqiswx-2 --reason ...`。 -->

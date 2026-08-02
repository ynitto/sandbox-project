---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqiswv-1
kind: plan-review
---

# 実行前レビュー: document-msbqiswv-1 — node-budget-summary スキーマを追加し status/<node>.json へ埋め込む

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : node-budget-summary スキーマを追加し status/<node>.json へ埋め込む
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. schemas/node-budget-summary.schema.json がリポジトリに追加されている
    2. status/<node>.json の budget キーが schema に準拠して出力される fixture がある
    3. 旧ビューが optional な新フィールドで壊れない契約テストが通る
- why: Phase1 の前提である射影 schema を先に固定し、互換性テストで安全に出すため。
- desc: schemas/node-budget-summary.schema.json を追加し、status/<node>.json の budget block に additive に埋める仕様を定義。reader が optional で壊れない互換性を担保する契約テストを追加する。
- charter: v1
- assess: c=2 r=2 a=2
- priority: 2
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqiswv-1`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqiswv-1 --reason ...`。 -->

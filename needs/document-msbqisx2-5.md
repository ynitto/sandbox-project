---
status: proposed
date: 2026-08-02
decision-makers: [human]
task-id: document-msbqisx2-5
kind: plan-review
---

# 実行前レビュー: document-msbqisx2-5 — 共有禁止項目（redaction）を契約テスト化する

## Context and Problem Statement

- なぜ: 新規タスクの実行前レビュー（承認されるまで実行しません）
- 状態: proposed（実行前レビュー待ち・未実行）

## タスク定義（レビュー対象）
- title  : 共有禁止項目（redaction）を契約テスト化する
- verify : （未定義）
- acceptance（受入基準・検証エージェントがこれを証跡付きで判定します）:
    1. プライバシー用 fixture を用いた redaction テストが追加されている
    2. テストにより token/ホームパス/生プロンプトが共有ファイルに現れないことが自動検出される
    3. redaction 失敗時に CI が失敗するようになっている
- why: Phase0 の必須安全網。共有前検査を自動化し漏出リスクを阻止する。
- desc: token / ホームディレクトリ / 生プロンプト / 生の資格情報が状態リポジトリや brief/decisions へ出ないことを fixture で固定する redaction テストを追加する。
- charter: v1
- assess: c=2 r=3 a=2
- priority: 5
- source : enqueue

## Decision Outcome

<!-- 人の決定の記入欄。承認は空のまま [x]、差し戻しは修正指示を書いて [x]。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 承認して実行を許可するなら `agent-project approve document-msbqisx2-5`（または空のまま [x]）。
     差し戻す（agent-project にタスクを修正させる）なら下に修正指示を書いて [x]。
     却下（廃止して関連バックログを再計画）なら `agent-project reject document-msbqisx2-5 --reason ...`。 -->

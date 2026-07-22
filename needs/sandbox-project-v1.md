---
status: proposed
date: 2026-07-23
decision-makers: [human]
task-id: sandbox-project-v1
kind: milestone
---

# マイルストーン: v1

## Context and Problem Statement

- なぜ: acceptance 未定義（done 判定不能→人へ）
- 状態: no-acceptance
- 概況: 自然文の完了条件を検証コマンドへ変換できませんでした（done 判定不能）: agent-project codd-gate 連携対応コードが個別対応ではなく将来別の CLI にも転用できること / agent-project の設計書とコードの一貫性が保たれていること ／ 対処: 検証コマンド・テンプレートで書き直すか、機械検証できない条件なら行頭に `検収:` を付けて人の検収項目にしてください（収束時に確認して承認できます）

## goal
* agent-project の codd-gate 連携コードを整理する
* agent-dashboard でのユーザビリティを考慮する

## Decision Outcome

<!-- 人の決定の記入欄（MADR の Decision Outcome）。方針・指示をここに書く。 -->
- [ ] 確定（このボックスを [x] にして保存すると取り込みます）

<!-- 完了として受領するなら `agent-project approve sandbox-project-v1 --reason ...`（プロジェクト done）。
     次フェーズへ続けるなら charter.md の goal/acceptance を更新して再実行。
     方向修正なら下に方針を書いて [x]（または policy.md を編集）。 -->

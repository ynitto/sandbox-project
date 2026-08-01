## dashboard-163827: dashboard で一貫性ゲートの状態把握と有効化を支援する
- status: done
- source: charter
- priority: 0
- verify: `echo "done"`
- retries: 18
- workspace: agent-dashboard
- refs: agent-project
- why: パッケージ内マジック配線を外した後も、人が regression/intake の有無とゲート失敗の意味を画面から判断・対処できるようにするため。
- out_of_scope: agent-project 本体のフック実装・done 不変条件を破る UI からの状態書換
- hints: 概要またはプロジェクト情報に regression_cmd/intake_cmd（設定の有無）を可視化し、未結線時は README と同じ有効化導線（設定編集／sibling CLI）を示す。needs の codd-gate / 回帰失敗要約（needs-diagnosis）の可読性を落とさない。公式契約（needs/inbox/commands）以外へ書かない。実装後は agent-reviewer で UX レビュー。
- charter: v1
- after: codd-gate-163827, sibling-163827
- assess: c=2 r=1 a=1
- rev: 2
- edited: human
- needs_reason: 繰り返し NG（retries=18）: agent-flow run タイムアウト（1800.0s）
- last_run: req-48d24769-dashboard-163827-r15-v2
- verification: {"pass": 2, "fail": 0, "unverifiable": 0, "report": "verifications/dashboard-163827/016a4bde9bf90b57d6cdc35571fcb17674079ab9.md", "receipt": true, "plan_digest": "sha256:146cdf038e5db43cfbc5b1b47abe53a3d7b205e1e7093a22b6ee54c96f1304bc"}
- needs_dr: DR-0018
- archived: 2026-08-02 04:49:21

## 納品書
- 完了 : 2026-08-02 04:49:21
- verify: `echo "done"` → PASS（基準 2 件中 2 件 pass（agent-flow runner の receipt を検算して採用））
- 成果 : commit 48d24769

## 判断材料（成果物の所在・差分・検証）
- 成果物: commit 48d24769
- 所在: /Users/nitto/Workspace/sandbox-project / ブランチ main

## run ブリーフ（この試行群で確定した制約・教訓。learn 射影済み）
- コンフリクトを解消して
- 現状のmainブランチが大幅に変わっているためrebaseして再度作業する。
- マージ先の main とコンフリクトしているため最新をpullして解消して
- コンフリクトを解消する
- 最新 main を統合し、6ファイルの競合を解消して全検証をやり直す。main由来のMarkdown末尾空白は競合として扱わない。
- 競合解決済み commit 59ccf49e を起点に、旧 run の done ノードを継承せず新規計画で全検証する。work の terminal ok:false は失敗として扱う。

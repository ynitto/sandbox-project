# agent-dashboard gate verification

## 判定

`verify=fail`

## blocking issues

1. **[t1 / functional] 実効グローバル設定を未設定扱いする**
   - 場所: `tools/agent-dashboard/src/features/agent-project/main/project.js:1609`
   - `consistencyGateStatus()` は `_configFromWorkspace()` により `~/.agents/agent-project.*`
     を捨てるが、agent-project 本体の
     `tools/agent-project/agent_project/configfile.py:184-203` はローカル設定が無い場合に
     `agent_home_dir()` を公式 fallback として採用する。
   - そのため本体が global `regression_cmd` / `intake_cmd` を実行していても dashboard は
     「設定なし・未結線」と誤表示し、不要なローカル設定作成を案内する。
   - 最小修正: project root 解決で global `root` を無視する既存方針とゲートの実効設定判定を
     分離し、`consistencyGateStatus()` は `readToolConfig()` が返した実効 `values/file` を採る。
     `test/consistency-gate.test.js:136` は global 設定を未結線とする期待を反転し、
     ローカル設定が global より優先されるケースも固定する。

2. **[t2 / test] 概要への実結線と有効化ボタン動作を落としてもテストが通る**
   - 場所: `tools/agent-dashboard/test/overview-ui.test.js:284,310-324`
   - `/consistencyGateHtml\(p\)/` は関数定義にも一致するため、`renderOverview()` の
     `${consistencyGateHtml(p)}` を削除しても成功する。また HTML の `data-gate-open` だけを確認し、
     `bindConsistencyGate(el)` と click 時の `api.openPath()` を検証していない。
   - 最小修正: call-site を `${consistencyGateHtml(p)}` として照合し、
     `bindConsistencyGate(el)` の存在を固定する。偽 root/button と `api.openPath` spy で、
     click が `data-gate-open` のパスを一度だけ渡すことを確認する。

## 独立検算

- classify: `class=investigation`。graph の t1/t2 scope と out-of-scope を再確認。
- コンフリクト: `git ls-files -u` と conflict marker 検索で残存なし。
- スコープ: `main...HEAD` の変更 12 ファイルはすべて `tools/agent-dashboard/` 配下。
- 件数: 差分は 12 ファイル、`+973/-7`。依存報告の「7ファイル」は現行差分と不一致。
- regression/intake: 設定有無・コマンド値・codd-gate 結線は別々に表現され、
  結線正規表現は公式 `codd_gate_wiring.py` と一致。global fallback だけ不一致。
- 未結線時: README と同形の設定行、設定ファイルを開く導線、regression 用 sibling CLI、
  intake は直接編集である旨を表示。UI 自身は設定を書き換えない。
- codd-gate / 回帰失敗: 構造化 frontmatter と旧記録 fallback の双方で回帰経路と task verify
  経路を区別し、既存の要約・context・判断材料の折り畳みを維持。
- needs/inbox/commands: 読み取り・表示追加のみで既存の push 型契約を変更していない。
- done 不変条件: ゲート UI の操作は `api.openPath` のみ。done 確定・状態遷移への新規書込みなし。

## agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---:|---:|---:|---:|
| functional | REQUEST_CHANGES | 0 | 1 | 0 |
| ai-antipattern | REQUEST_CHANGES | 0 | 1 | 0 |
| architecture | LGTM | 0 | 0 | 1 |
| test | REQUEST_CHANGES | 0 | 1 | 0 |

総合判定: Request Changes

architecture の suggestion は非 blocking:
`consistencyGateHtml` / `bindConsistencyGate` は overview 専用なので、次回変更時に
`sections/overview.js` へ寄せると既存分割境界に揃う。

## 検証結果

- 指定 grep: exit 0
- `needs-diagnosis.test.js`: exit 0（13/13）
- `overview-ui.test.js`: exit 0
- `consistency-gate.test.js`: exit 0（7/7。ただし blocking issue 1 の誤契約を固定）
- `consistency-gate-ui.test.js`: exit 0
- `needs-gate-integration.test.js`: exit 0（8/8）
- `npm test`: exit 0
- `git diff --check main...HEAD`: exit 0
- worktree: clean

@followup t1: `consistencyGateStatus()` を agent-project の実効 global fallback と一致させ、
global/local 優先順位テストを修正する。

@followup t2: overview の実 call-site と `bindConsistencyGate` → `api.openPath` のクリック配線を
退行検知するテストへ最小修正する。

{"ok": false, "issues": ["[t1] tools/agent-dashboard/src/features/agent-project/main/project.js:1609 — agent-project 本体が採用する ~/.agents の実効 regression_cmd/intake_cmd を捨てるため、結線済みでも未設定・未結線と誤表示する。consistencyGateStatus は readToolConfig の実効 values/file を採り、global/local 優先順位テストを修正する。", "[t2] tools/agent-dashboard/test/overview-ui.test.js:284,310-324 — consistencyGateHtml の関数定義だけで call-site 検査が通り、data-gate-open の click 配線も未検証。${consistencyGateHtml(p)} と bindConsistencyGate(el) を固定し、api.openPath spy のクリックテストを追加する。"]}

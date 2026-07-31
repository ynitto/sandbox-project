差別化の切り口: 設定値と失敗記録の二つの読み取り経路を分け、画面が断定できる範囲を明示した。

# t7 agent-dashboard 設計書更新

## 成果／サマリー

完了条件を「t3 が特定した `tools/agent-dashboard/README.md` の既存『一貫性ゲート』節だけを更新し、公式契約から概要・要対応までの経路、未結線時の有効化手順、読み取り専用境界を短く説明すること」とした。

同節に次を反映した。

- `agent-project.yaml` の `regression_cmd` / `intake_cmd` から `readProject().consistencyGate`、IPC、概要表示へ至る経路。
- `needs/<id>.md` の `failure-*` から要対応の失敗表示へ至る経路。現在の設定ではなく、失敗時の phase、要約、コマンドを根拠に codd-gate 由来を判定する。
- 未結線時は YAML 編集を案内し、`regression_cmd` に限って sibling CLI を提示する。dashboard 自身は CLI を実行しない。
- dashboard は設定、タスク状態、done を書き換えず、「設定ファイルを開く」操作も OS のエディタを開くだけとする境界。

slop-police に従って結論を冒頭へ置き、実装名で反証できる記述に絞った。新しい文書や章立ては追加していない。

## 検証内容と結果

- `npm test`: exit 0、全件成功。
- `git diff --check`: 成功。
- 記載した `readToolConfig`、`consistencyGateStatus`、`dashboard:project`、`parseNeeds`、`renderNeedFacts`、`consistencyGateHtml` が対象実装に存在し、t3〜t5 の報告と一致することを確認した。
- `git diff --name-only`: `tools/agent-dashboard/README.md` だけ。許可範囲外の変更なし。

## 前提・未解決事項・範囲外

- 前提: t3 の「README: 設定編集、sibling CLI、読み取り専用という導線を UI と同期させる」を、既存設計書 `tools/agent-dashboard/README.md` の指定と解釈した。
- 前提: t4 と t5 が検証した現行実装を正とし、設計書をその実装へ合わせた。
- 未解決事項なし。
- agent-project のフック仕様、dashboard の実装、done 不変条件、他の文章は変更していない。

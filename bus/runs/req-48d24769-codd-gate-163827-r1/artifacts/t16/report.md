切り口: 設定例を境界の証拠にし、「検出」「永続化」「有効化」の主体を続けて読める形にした。

## 成果

`tools/agent-project/README.md` の codd-gate 連携節を正典
`docs/designs/codd-gate-design.md` §4、§4.1 に合わせた。

- `regression_cmd` と `intake_cmd` の YAML 例を並べ、対応 CLI でも人か install 手順による明示設定だけが
  連携を有効にすると明記した。
- 任意 sibling 部品の自動検出対象を、実体、バージョン、repos schema 互換性、対応機能と明記した。
- `codd_gate_regression.py` の永続化責務を `.agent/agent-project.yaml` の
  `regression_cmd` 1 行の冪等書き込みに限定し、`intake_cmd` は書かないと明記した。

変更は README 1 ファイルだけ。agent_project、dashboard、テスト、設計書には触れていない。

## 検証

- `git diff --check`: PASS。
- `git diff --name-only`: `tools/agent-project/README.md` のみ。
- README の同じ節に、両設定例、自動検出の4対象、`regression_cmd` だけを永続化する責務、
  自動検出だけでは有効にならない条件があることを `rg` で確認した。
- 文書だけの変更なので、コードテスト、リンタ、型チェックは実行していない。

## 前提・未解決事項

- 完了条件は、README 単体で「sibling は値を検出・生成する任意部品」「有効化は明示設定」
  「永続化ツールが書くのは `regression_cmd` だけ」を一続きに確認できること、と解釈した。
- `codd_gate_*.py` の自動検出は起動時の Config 自動配線ではなく、人か install 手順が任意部品を
  明示起動したときの検出を指す、とした。
- 範囲外の既知問題: README に記載された境界ゲート
  `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  は現時点で失敗する。`agent_project/configfile.py`、`doctor.py`、`model.py` に該当実装が残っている。
  今回は実装変更とテスト改修が禁止されているため修正していない。

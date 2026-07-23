# README「一貫性ゲート」現行記述の抽出

対象: `tools/agent-project/README.md` 272–289行

## 抽出結果

- 責務境界（272–275行）: ドキュメント、コード、テストの整合は完全独立の `codd-gate` が担う。agent-project との結合は共通スキーマだけであり、agent-project が charter から生成した `<root>/repos.json` を codd-gate が `--repos` で読む。
- `regression_cmd`（275–278行）: `regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'` を、done 確定前の差分ゲートとして設定する。
- `intake_cmd`（275–278行）: `intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'` を、検出した負債を修復タスクとして自動返済するために設定する。charter acceptance の `codd-gate verify --debt --max-broken N …` は受入時の負債ラチェットを担う。
- `codd_gate_*.py`（279–287行）: `agent_project` パッケージは `codd_gate_*` を import、結合、依存しない。`codd_gate_regression.py` は `python3 codd_gate_regression.py --config .agent/agent-project.yaml` で YAML に設定を冪等注入する。`codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品であり、欠落または削除してもパッケージの挙動は変わらない。codd-gate が未検出または非互換なら、生成ツールは値を書かず no-op に縮退する。
- 自動検出（282–283行）: 起動時の Config 生成 `build_config` が codd-gate を自動検出し、設定値を差し込む配線層はない。手動設定がなければ値は空のままになり、連携なしとして通過する。
- 有効化方法（275–285行）: 有効化は設定だけで行う。人または install 手順が `regression_cmd` と `intake_cmd` を YAML または CLI に書く。永続化が必要なら、人が `codd_gate_regression.py` を起動する。`.agent/agent-project.yaml` は人専有ファイルであり、機械による自動書き換えはしない。

## 検証

- `nl -ba tools/agent-project/README.md | sed -n '262,290p'` 相当で、対象節と行番号を照合した。
- 指定された6観点をすべて抽出した。リポジトリ内のファイルは変更していない。

## 前提と未解決事項

- 「一貫性ゲート節」は、README 272行から289行までの単一箇条書きを指すと解釈した。
- 「自動検出」は自動検出機能の存在ではなく、現行文書が明記する「自動検出配線はない」という境界として抽出した。
- 未解決事項および範囲外で見つけた問題はない。

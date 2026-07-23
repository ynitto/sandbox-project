# t20 成果報告

## サマリー

`tools/agent-project/README.md` には loop と t18 の変更が共存しており、追加修正は不要だった。
リポジトリ内ファイルは変更していない。

判定は `verify=fail`。`docs/designs/codd-gate-design.md` §4.2 の必須ゲートが
`_apply_codd_gate` だけを検査し、`_codd_gate` と `import codd_gate` を任意例にしているため、
要求された `verify=pass` の条件を満たさない。

## 検証

- README と設計書 §4・§4.1: PASS。パッケージは汎用フックだけ、`codd_gate_*.py` は任意 sibling と記載されている。
- E1–E3: PASS。E2 の `codd-gate verify --base "$KIRO_BASE_REV" ...`、E3 の
  `codd-gate tasks --debt ...`、E1 acceptance の `codd-gate verify --debt ...` と
  task verify の `codd-gate check …` を確認した。
- 有効化と永続化: PASS。YAML/CLI 設定だけで有効化し、`codd_gate_regression.py` だけが
  `regression_cmd` 1行を永続化すると記載されている。
- §4.2: FAIL。必須コマンドは
  `! git grep _apply_codd_gate -- tools/agent-project` のまま。
  要求相当の正規表現を package に実行すると14件を検出し、自動配線、private 名、固有 import が
  現実装に残ることは確認できた。
- 元要求の4段連結シェルコマンド: exit 0。
- `git diff --check` と `git diff --check main...HEAD`: exit 0。
- `main...HEAD` の変更範囲: `docs/designs/codd-gate-design.md` と
  `tools/agent-project/README.md` の2文書だけ。`agent_project/`、dashboard、テストの差分は0。
- 作業ツリー: clean。

## 前提・未解決事項・範囲外

- 「競合解消」は、現行 target で loop と t18 の README 変更が失われていないことを確認し、
  必要なら許可範囲内だけを直すこと、と解釈した。worktree は detached HEAD かつ clean で、
  rebase の競合マーカーや未解決 index は存在しなかった。
- 書込許可 `tools/agent-project/` を優先した。正典は許可範囲外なので、
  t18 の `codd-gate-design.patch` を直接適用していない。
- 未解決: 設計書の所有者が §4.2 の必須ゲートを
  `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  相当へ変更する必要がある。
- 範囲外の実装、dashboard、テストには触れていない。

@followup docs-design-owner: `docs/designs/codd-gate-design.md` §4.2 に t18 のパッチを適用し、同じ検証を再実行する。

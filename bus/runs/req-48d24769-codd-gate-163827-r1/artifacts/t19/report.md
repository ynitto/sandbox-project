# t19 成果報告

## 成果

`docs/designs/codd-gate-design.md` §4、§4.1、§4.2 を slop-police の観点で点検し、
責務主体、許可事項、禁止事項を同じ表で読める修正文案を
[`proposed-edit.md`](proposed-edit.md) にまとめた。

主な修正点は次のとおり。

- `agent_project/*`、人または install 手順、任意 sibling 部品、codd-gate CLI を主体として明記した。
- パッケージの禁止境界を「名指し、import、検出、推奨値生成、起動時の自動配線」と具体化した。
- yaml の書き手を、人または install 手順が明示実行した
  `codd_gate_regression.py` に限定した。
- 完了条件を、依存成果 t17 の三つの検出語と同じパッケージ境界を使う一つの否定コマンドに統合した。

正典そのものは変更していない。指定された書込可能範囲が `tools/agent-project/` 配下だけで、
対象の `docs/designs/codd-gate-design.md` はその外にあるためである。

## 検証

- slop-police の優先項目である主体性、false agency、具体性、削減の順に修正文案を確認した。
  「誰が書くか」「誰が呼ぶか」「パッケージがしてはいけないこと」を各段落で明記している。
- 依存成果 t17 の候補コマンドを現 HEAD で実行した。

  ```sh
  ! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
  ```

  結果は exit 1。`configfile.py`、`doctor.py`、`model.py` に禁止語が残っており、未完了を正しく検出した。
- `git status --short --untracked-files=all` は出力なし。リポジトリの作業ツリーは変更していない。
- 文書案のみのため、テスト、リンタ、型チェックは実行していない。

## 前提・未解決事項・範囲外

- 完了とは、正典に反映した後で「責務主体と禁止境界が一読で分かり、三つの禁止語を一つの
  パッケージ境界で機械判定できること」と解釈した。
- §4.1 の自動検出はパッケージ起動時の自動配線ではなく、人または install 手順が任意部品を
  明示実行したときの検出を指す、と解釈した。
- 対象文書への未反映だけが未解決。修正文案は自己完結している。
- agent-project、dashboard、テストの実装変更と、検出された残存コードの削除は範囲外。

@followup `docs/designs/codd-gate-design.md` を書込可能範囲に含むタスクで
`proposed-edit.md` を反映し、同じ否定コマンドが exit 0 になる実装タスクを別に実施する。

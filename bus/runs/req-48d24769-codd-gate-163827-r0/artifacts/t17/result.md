# t17 検証結果

verify=fail

## 完了条件コマンド

`meta.json` 記載の次のコマンドを、指定 worktree 直下で原文どおり一度実行した。

```sh
grep -nE 'agent_project.*(import|結合|依存).*(しない|外|禁止)|パッケージ.*(codd_gate|sibling)|有効化は設定' tools/agent-project/README.md && grep -nE 'regression_cmd|intake_cmd|codd_gate_\*\.py|自動検出' tools/agent-project/README.md && test -f docs/designs/codd-gate-design.md && grep -nE 'agent_project パッケージ|_apply_codd_gate|sibling|汎用フック' docs/designs/codd-gate-design.md
```

- コマンド全体: exit 0
- grep 1: exit 0、未一致パターンなし
- grep 2: exit 0、未一致パターンなし
- test -f: exit 0、対象ファイルあり、標準出力なし
- grep 3: exit 0、未一致パターンなし

`&&` の全段が実行され、全体が exit 0 だったため、各段の exit code はすべて 0 と確定した。

## 該当行

grep 1 (`tools/agent-project/README.md`):

```text
275:  `<root>/repos.json` を codd-gate が `--repos` で読む。**有効化は設定だけ**:
282:  `agent_project/*` パッケージは E1〜E6 の汎用フックだけを提供し、codd-gate を名指しせず、`codd_gate_*` を import・結合・依存しない。
287:  パッケージは sibling 部品を探索・import せず、`build_config` から値を差し込む自動配線も持たない。
```

grep 2 (`tools/agent-project/README.md`):

```text
276:  `regression_cmd` には
278:  確定前に差分の一貫性を検査し、NG なら done を止める。`intake_cmd` には
283:  `regression_cmd`/`intake_cmd` の有効化は、人か install 手順が yaml/CLI に書く場合に限る。永続化は sibling の
285:  冪等注入する。`codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品で、人か install 手順が
286:  明示起動したときだけ codd-gate の実体、バージョン、schema 互換性、対応機能を自動検出する。
437:- **取り込みコマンド（intake_cmd）**: 外部の決定的ゲート/検出器を **watch の周期で pull** する汎用フック（push 型の
438:  inbox と対）。設定 `intake_cmd:`（CLI `--intake-cmd`）のコマンドをパス開始時と idle 中に `intake_interval`（既定
442:  持つ）。例: `intake_cmd: codd-gate tasks --debt`（doc/code/test 一貫性の負債を修復タスク化して自動返済）。
443:  > 外部 CLI を差し込める公式の口（verify/acceptance・regression_cmd・intake_cmd・inbox/enqueue・
```

grep 3 (`docs/designs/codd-gate-design.md`):

```text
4:> `tools/agent-project/`（`codd_gate_detect.py` 等の任意 sibling 部品含む）, `docs/designs/agent-project-design.md`
248:- **パッケージ（`agent_project/*`）は汎用フックだけを提供する。** E1〜E6 の差し込み点を用意するのが責務で、
251:  codd-gate は「汎用フックへ渡されうる任意の値」であって、コード上の依存先ではない。
252:- **`codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品。** 標準ライブラリだけで動き、
255:  トップレベルの sibling モジュールとしてだけ存在する。丸ごと削除してもパッケージは同一挙動を保つ。
259:  この一意の書き手が汎用フックへ置く文字列値だけ。
277:`tools/agent-project/` 直下に **sibling として**置いてある（標準ライブラリのみ・agent-project.py 側の型
356:§4 の境界、すなわち「パッケージは汎用フックだけを提供し codd-gate を名指し・import・自動配線しない／
357:codd_gate_* は任意の sibling 部品」を、散文の宣言ではなく機械可読な受入で固定する。設計上の完了条件は、
361:! git grep _apply_codd_gate -- tools/agent-project
365:「非 0 を期待する」否定）がこれを反転させるので、`_apply_codd_gate` が1件も残っていないときにだけ全体が
366:exit 0（PASS）になる。この関数（`agent_project/configfile.py` の `_apply_codd_gate_auto_wiring`）はパッケージ内に
367:しか現れず、sibling の `codd_gate_*.py` や tests には出ない（それらは `resolve_codd_gate` などを正当に持つ）。
377:パスを `tools/agent-project` 全体へ広げると、sibling の `resolve_codd_gate` や tests の `import codd_gate_debt`
380:`_apply_codd_gate` は `_codd_gate` に包含されるため連結は冗長でもある）。コメントや docstring 中の `codd-gate`
385:`_apply_codd_gate_auto_wiring` と doctor/model の `import codd_gate_*` を除いた状態にすれば PASS する。除去そのものは
```

## 独立検査

- `git status --short`: 出力なし（未コミット変更なし）
- `git diff --check main...HEAD`: exit 0
- `git diff --name-status main...HEAD`: `docs/designs/codd-gate-design.md` と `tools/agent-project/README.md` の2件。重複なし。
- README の責務境界、有効化、永続化、sibling 自動検出、および正典 §4・§4.1・§4.2 を抜き取り確認し、完了条件に必要な内容は整合していた。

## issue

- `docs/designs/codd-gate-design.md` が `main...HEAD` で変更されているが、この worktree の変更許可範囲は `tools/agent-project/**` のみ。文書変更を `docs/designs/**` を所有する専用タスク/worktree へ移すか、書込許可範囲を明示的に拡張してから取り込むこと。現在のブランチのままではスコープ条件を満たさない。

{"ok": false, "issues": ["docs/designs/codd-gate-design.md が main...HEAD で変更されているが、許可範囲は tools/agent-project/** のみ。文書変更を docs/designs/** を所有する専用タスク/worktree へ移すか、書込許可範囲を明示的に拡張してから取り込むこと。"]}

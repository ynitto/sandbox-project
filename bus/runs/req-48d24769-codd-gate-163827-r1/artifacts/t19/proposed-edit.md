# `codd-gate-design.md` 修正文案

書込許可が `tools/agent-project/` 配下に限定されているため、正典には未反映。以下を
`docs/designs/codd-gate-design.md` の該当箇所へ反映する。

## §4「プラグイン境界（規範）」の箇条書きを置換

**プラグイン境界（規範）**。責務と禁止事項を次の表に固定する。

| 主体 | 責務 | 許可すること | 禁止すること |
|---|---|---|---|
| `agent_project/*` パッケージ | E1〜E6 の汎用フックを提供し、設定された外部 CLI を実行する | `regression_cmd`、`intake_cmd`、acceptance へ利用者が設定した文字列を、そのまま汎用契約で扱う | codd-gate の名指し、`codd_gate_*` の import、実体検出、推奨値の生成、起動時の自動配線 |
| 人または install 手順 | 連携を有効化するか決め、yaml または CLI に値を設定する | `codd_gate_regression.py` を明示的に実行し、`regression_cmd` を yaml へ永続化する | パッケージ起動に伴う暗黙の設定変更 |
| `tools/agent-project/codd_gate_*.py` | codd-gate の検出、使用可否の判定、実引数の組み立て、明示実行時の yaml 更新を行う | `tools/agent-project/` 直下の任意 sibling 部品として、標準ライブラリだけで単独動作する | `agent-project.py` 側の型への依存、`agent_project/*` からの import、パッケージ起動時の値注入 |
| codd-gate CLI | 渡された引数で単発の検査またはタスク生成を行う | `schemas/` の共通契約を介して agent-project と連携する | agent-project の実装、charter、常駐ループへの依存 |

`regression_cmd`、`intake_cmd`、acceptance の値を書く主体は、人または install 手順だけである。
`codd_gate_regression.py` は、その主体が明示実行したときに限り
`regression_cmd` を yaml へ書く。`build_config` を含むパッケージの起動経路は値を書かない。
`codd_gate_*.py` をすべて削除しても、未設定時のパッケージの挙動は変わらない。

## §4.1 の見出しと冒頭を置換

### 4.1 人または install 手順が使う任意 sibling 部品（`tools/agent-project/codd_gate_*.py`）

人または install 手順は、表①〜③の文字列を yaml か CLI に直接設定する。手入力の代わりに
検出、判定、実引数の組み立てが必要な場合だけ、`tools/agent-project/` 直下の
`codd_gate_*.py` を明示的に呼ぶ。`agent_project/*` パッケージはこれらを import も実行も
しない。任意部品は「実体の検出」「使用可否の判定」「実引数の組み立て」を順に行い、
`codd_gate_regression.py` だけが、明示実行時に `regression_cmd` を yaml へ保存する。

## §4.1「有効化」から「任意部品の可搬性」までを置換

**有効化の主体**: 人または install 手順が、`regression_cmd`、`intake_cmd`、acceptance の値を
yaml か CLI に書く。値が無ければ連携は無効である。`agent_project/*` パッケージは、
環境の検出結果から値を補わない。

**永続化の主体**: 人または install 手順が
`python3 tools/agent-project/codd_gate_regression.py --config .agent/agent-project.yaml`
を明示実行した場合だけ、`codd_gate_regression.py` が `regression_cmd` を一行更新する。
同ツールは codd-gate の実在、バージョン、schema、capability を確認し、使用できなければ
何も書かない。パッケージの起動経路は yaml を更新しない。

**人専有ファイルの禁止境界**: `.agent/agent-project.yaml` は
`agent_project/state.py` の `_HUMAN_OWNED_STATE_FILES` に含まれる。状態 worktree の
鏡合わせと `agent_project/*` パッケージは、このファイルを書き換えない。書き換えてよいのは、
人の直接編集と、人または install 手順が明示実行した `codd_gate_regression.py` だけである。

**任意部品の欠落時**: `codd_gate_*.py` を置かなくても、または後から削除しても、
`agent_project/*` パッケージの挙動は変わらない。パッケージは任意部品を import せず、
未設定の汎用フックを従来どおり通過する。手書き値を残すか削除するかは利用者が決める。

## §4.2 全文を置換

### 4.2 パッケージ境界の完了条件

`agent_project/*` パッケージには、codd-gate の自動配線、名指しした内部関数、import を残さない。
`tools/agent-project/` 直下の `codd_gate_*.py` は許可する。完了条件は、次の一行が exit 0 を
返すことである。

```sh
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

検索先をパッケージ本体に限定するため、任意 sibling 部品とそのテストは違反にならない。
`_apply_codd_gate`、`_codd_gate`、`import codd_gate` のいずれかがパッケージ内に残ると
`git grep` が exit 0 を返し、先頭の `!` が受入を失敗させる。三つとも無ければ受入は通る。

このゲートが固定する禁止事項は、パッケージからの自動配線、名指し、import である。
コメントや docstring を含む `codd.?gate` の総当たりは、コード結合の禁止を越えるため行わない。
残存コードの削除は実装タスクで扱う。

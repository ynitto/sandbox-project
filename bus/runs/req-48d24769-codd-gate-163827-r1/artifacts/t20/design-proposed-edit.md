# `codd-gate-design.md` 反映案

書込許可が `tools/agent-project/` 配下に限られるため、正典には未反映。以下を
`docs/designs/codd-gate-design.md` §4〜§4.2 へ反映する。

## §4「プラグイン境界（規範）」の箇条書きを置換

**プラグイン境界（規範）**では、責務を次の表に固定する。

| 主体 | 責務 | 禁止事項 |
|---|---|---|
| `agent_project/*` パッケージ | E1〜E6 の汎用フックを提供し、利用者が設定した外部 CLI を実行する | codd-gate の名指し、`codd_gate_*` の import、実体検出、推奨値生成、起動時の自動配線 |
| 人または install 手順 | 連携を有効化するか決め、yaml または CLI に値を設定する | パッケージ起動に伴う暗黙の設定変更 |
| `tools/agent-project/codd_gate_*.py` | codd-gate の検出と実引数の組み立てを補助し、明示実行時だけ yaml を更新する | `agent_project/*` からの import、パッケージ起動時の値注入 |
| codd-gate CLI | 渡された引数で検査または修復タスク生成を行う | agent-project の実装、charter、常駐ループへの依存 |

コマンド名と差し込み点は次のとおり。

| 差し込み点 | コマンド | 完了判定での役割 |
|---|---|---|
| E1 task verify | `codd-gate check …` | 修復タスクの期待状態を確認する |
| E1 charter acceptance | `codd-gate verify --debt --max-broken N …` | プロジェクト受入時の負債ラチェットを判定する |
| E2 `regression_cmd` | `codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json` | verify PASS 後、done 確定前の横断検査を行う |
| E3 `intake_cmd` | `codd-gate tasks --debt --repos <root>/repos.json` | 負債を修復タスクへ変換して backlog に取り込む |

`regression_cmd`、`intake_cmd`、acceptance の値を設定する主体は、人または install 手順である。
`codd_gate_regression.py` は、その主体が明示実行したときに限り `regression_cmd` を yaml へ書く。
`build_config` を含むパッケージの起動経路は値を書かない。`codd_gate_*.py` をすべて削除しても、
未設定時のパッケージの挙動は変わらない。

## §4.1 の見出しと有効化・永続化を置換

### 4.1 人または install 手順が使う任意 sibling 部品（`tools/agent-project/codd_gate_*.py`）

人または install 手順は、§4 のコマンド文字列を yaml か CLI に直接設定する。手入力の代わりに
検出と実引数の組み立てが必要な場合だけ、`tools/agent-project/` 直下の `codd_gate_*.py` を
明示的に呼ぶ。`agent_project/*` パッケージはこれらを探索、import、実行しない。

連携を有効化するのは、人または install 手順である。`regression_cmd`、`intake_cmd`、acceptance の
値を yaml か CLI に書く。値が無ければ連携は無効で、`agent_project/*` パッケージが環境の検出結果から
補うことはない。

`regression_cmd` を永続化するには、人または install 手順が
`python3 tools/agent-project/codd_gate_regression.py --config .agent/agent-project.yaml`
を明示実行する。同ツールは `regression_cmd` を一行だけ更新する。現行実装の `detect_status()` が
確認するのは codd-gate の実体だけで、バージョン、repos schema 互換性、capability は検査しない。
実体を検出できなければ値を書かない。

`.agent/agent-project.yaml` は人専有ファイルである。人の直接編集と、人または install 手順が
明示実行した `codd_gate_regression.py` だけが書き換えてよい。状態 worktree の鏡合わせと
`agent_project/*` パッケージは書き換えない。

`codd_gate_*.py` を置かなくても、後から削除しても、`agent_project/*` パッケージの挙動は変わらない。
未設定の汎用フックは従来どおり通過する。

## §4.2 全文を置換

### 4.2 パッケージ境界の完了条件

`agent_project/*` パッケージには、codd-gate の自動配線、名指しした内部関数、import を残さない。
`tools/agent-project/` 直下の `codd_gate_*.py` は許可する。完了条件は、次の一行が exit 0 を
返すことである。

```sh
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

検索先をパッケージ本体に限定するため、任意 sibling 部品、テスト、README は違反にならない。
いずれかの禁止語がパッケージ内に残れば受入は失敗し、すべて無ければ通る。
コメントや docstring を含む `codd.?gate` の総当たりは、コード結合の禁止を越えるため行わない。
残存コードの削除は実装タスクで扱う。

@followup `codd_gate_regression.py` の永続化前判定を `detect_wiring()` に統合し、
バージョン、repos schema 互換性、capability の不適合時に値を書かない受入を追加する。

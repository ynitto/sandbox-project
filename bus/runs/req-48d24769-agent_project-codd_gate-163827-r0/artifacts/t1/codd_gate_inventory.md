# agent_project 内 codd_gate 依存調査

## サマリー

調査対象は指定どおり `tools/agent-project/agent_project/**/*.py` の現在の作業ツリーとした。
この範囲には次のいずれも存在しない。

| 区分 | 件数 | 一覧 |
|---|---:|---|
| module 名または import 対象名に `codd_gate` を含む import | 0 | なし |
| `_apply_codd_gate*` の定義 | 0 | なし |
| `_apply_codd_gate*` の呼び出し元 | 0 | なし |
| `_codd_gate*` の定義 | 0 | なし |
| `_codd_gate*` の呼び出し元 | 0 | なし |
| 上記識別子の非呼び出し参照 | 0 | なし |

したがって、指定された定義に対する「全呼び出し元」は空集合である。参考情報に挙げられた
`configfile._apply_codd_gate_auto_wiring`、`doctor._codd_gate_wiring_module`、
`doctor_codd_gate_findings` も対象範囲には存在しない。

## 残存する `codd_gate` / `codd-gate` 文字列

依存・専用識別子ではないが、取りこぼし判定用に文字列、コメント、docstring の全8箇所を
ファイル・関数単位で列挙する。

| ファイル:行 | 所属関数 | 種別・内容 |
|---|---|---|
| `agent_project/charter.py:373` | `export_repo_registry` | docstring: 外部ツール例 |
| `agent_project/configfile.py:145` | module scope (`DEFAULTS`) | コメント: `intake_cmd` の外部ゲート例 |
| `agent_project/configfile.py:625` | `_add_common` | CLI help: `intake_cmd` のコマンド例 |
| `agent_project/model.py:497` | `_parse_intake_records` | docstring: 実装非依存である旨と外部ゲート例 |
| `agent_project/model.py:543` | `run_intake` | docstring: 汎用フックの入力例 |
| `agent_project/mr.py:580` | `_settle_task` | コメント: regression 実行 cwd の理由を示す具体例 |
| `agent_project/verify.py:9` | `run_verify` | コメント: shell 連鎖中の無出力コマンド例 |
| `agent_project/verify.py:356` | module scope (`_SHELL_COMMANDS`) | shell コマンド抽出用の許可語彙 |

## 現在の汎用差し込み点（後続作業向け参考）

指定識別子は既に汎用名へ整理されている。直接の対応関係を履歴から断定したものではなく、
現在のコード上で同じ責務領域にある箇所である。

| ファイル | 定義 | 呼び出し元 |
|---|---|---|
| `agent_project/hooks.py` | `_hook_provider` | `hooks.py:_hook_resolution_error`; `doctor.py:doctor_wiring_findings` |
| `agent_project/configfile.py` | `_normalize_hooks` | `configfile.py:build_config` |
| `agent_project/doctor.py` | `_hook_misconfig_findings` | `doctor.py:doctor_wiring_findings` |
| `agent_project/doctor.py` | `doctor_wiring_findings` | `doctor.py:cmd_doctor` |

## 検証

- `rg` で対象配下を exact/prefix 検索し、該当 import・定義・呼び出しが0件であることを確認した。
- 全 Python ファイルを `ast.parse` し、`Import` / `ImportFrom`、`FunctionDef` /
  `AsyncFunctionDef`、`Call`、`Name` を独立に走査した。結果は import 0、定義 0、呼び出し 0、
  その他参照 0。
- 大文字小文字を無視した全文検索で残存文字列が8件のみであることを確認し、上表と照合した。
- 調査タスクのためコード変更・テスト実行は不要と判断した。指定 worktree 内のファイルは変更していない。

## 前提・未解決事項・範囲外

- 「配下」は `tools/agent-project/agent_project` ディレクトリ以下に限定し、同階層の
  `codd_gate_*.py` と `tests/test_codd_gate_*.py` は調査件数に含めていない。
- 「import」は Python AST 上の import 文を指す。動的 import も見落とさないよう全文検索を
  併用したが、対象範囲に `codd_gate` を含む動的 import 文字列はない。
- 現在のブランチでは除去/改名対象が既に消えているため、旧定義の過去の呼び出し関係は一覧の
  対象外とした。履歴上の旧構造が必要なら別途 git 履歴調査が必要。
- 範囲外の sibling `tools/agent-project/codd_gate_*.py` とそのテストには codd-gate 固有実装が
  多数残るが、元要求で本体外のプロバイダとして許容される領域であり、変更していない。

@followup 旧版からの改名対応表が必要なら、後続タスクで git 履歴を指定して旧定義・旧呼び出し元を調査する。

切り口: E1〜E3を「`agent_project` が提供する汎用契約」と「任意 sibling が渡す codd-gate 固有値」の対照表にする。

## 成果候補

`docs/designs/codd-gate-design.md` §4 の「プラグイン境界（規範）」直後から表④までを、次の記述へ更新する。

```markdown
**プラグイン境界（規範）**。この節が定めるのは目標状態であり、現 HEAD の整理完了を示すものではない。

- **パッケージ（`agent_project/*`）は E1〜E6 の汎用フックだけを提供する。**
  パッケージのコードから codd-gate を名指し、import、自動配線しない。フックの値が未設定なら空のまま扱う。
- **`codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品である。**
  パッケージから import されず、削除しても `agent_project` の挙動を変えない。codd-gate 固有の検出や
  コマンド生成はこの sibling 側に閉じる。

| 差し込み点 | `agent_project` の汎用フック | sibling 側の codd-gate 固有処理 | 有効化する主体 |
|---|---|---|---|
| E1 タスク `verify` / charter `## acceptance` | タスク spec の `- verify:` を done 判定に使い、charter の `## acceptance` をプロジェクト受入判定に使う。コマンドの内容は解釈しない | codd-gate が生成する修復タスクは `codd-gate check …` を `verify` に含められる。負債ラチェットを採用するプロジェクトは `codd-gate verify --debt --max-broken 0 …` を charter に書ける。`codd_gate_*.py` は E1 の値を自動注入しない | タスク生成元またはタスク作者が `verify` を、charter 作者が `acceptance` を書く |
| E2 `regression_cmd` | YAML または CLI で受け取った任意のコマンドを、毎タスクの verify PASS 後、done 確定前に実行する。未設定なら何もしない | `codd_gate_regression.py` を人か install 手順が明示実行すると、`codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json` を YAML の `regression_cmd` 1 行へ冪等注入する。起動時の自動検出や自動配線はしない | 人または install 手順。機械的に永続化する sibling は `codd_gate_regression.py` だけ |
| E3 `intake_cmd` | YAML または CLI で受け取った任意のコマンドを `intake_interval` ごとに実行し、共通 task スキーマの出力を backlog へ冪等取り込みする。未設定なら何もしない | `codd-gate tasks --debt [--cohort]` を設定すれば負債から修復タスクを供給できる。`intake_cmd` を生成または YAML へ永続化する `codd_gate_*.py` は置かない | 人または install 手順が YAML/CLI に設定する |
| （補）repos レジストリ（`schemas/repos.schema.json`） | charter から `<root>/repos.json` を生成する | codd-gate は `--repos` で読む。charter は読まない | 共通データ契約 |

`$KIRO_BASE_REV` は agent-project が verify / regression に渡す act 前 HEAD をそのまま使う。
ワークスペース運用では、タスク生成時に `--repo-dir <name>=.` を焼き込むことで clone 内で自己完結する。
```

この差し替えでは、E1 を YAML/CLI 有効化の対象から外した。E1 の保存先は task spec と charter である。
E2 と E3 は同じ汎用設定フックだが、sibling の永続化対象は E2 の `regression_cmd` だけである。

## 検証

- `docs/designs/codd-gate-design.md` §4 と §4.1、`tools/agent-project/codd_gate_regression.py`、
  `tools/agent-project/codd_gate_wiring.py`、依存成果 `synth1/report.md` を突合した。
- E1: task `verify` と charter acceptance の保存先を分離し、sibling が自動注入しないことを明記した。
- E2: 汎用の実行契約と、`codd_gate_regression.py` が `regression_cmd` 1 行だけを永続化する処理を分離した。
- E3: 汎用の周期取り込み契約と、codd-gate 固有コマンドを人か install 手順が設定する処理を分離した。
- 作業ツリーで `git status --short` を確認し、変更がないことを確認した。文書候補のみなのでテストは実行していない。

## 前提・未解決事項・範囲外

- 完了条件は、E1〜E3 の各行だけで汎用フック、codd-gate 固有処理、有効化主体を判別できることとした。
- 対象文書はリポジトリ直下の `docs/designs/codd-gate-design.md` にあり、書込許可された
  `tools/agent-project/` 配下ではない。書込制約を優先し、作業ツリーには適用していない。
- 現 HEAD の `agent_project/configfile.py`、`doctor.py`、`model.py` に残る codd-gate 専用結合は範囲外であり、
  修正していない。

@followup 書込権限を持つ後続タスクで、この候補を `docs/designs/codd-gate-design.md` §4 に適用する。
@followup 別タスクで `agent_project` 内の専用結合と関連テストを汎用フック境界へ合わせる。

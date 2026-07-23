# t4 成果報告

## 成果

対象の `docs/designs/codd-gate-design.md` は、書き込みを許可された `tools/agent-project/` 配下に存在しない。範囲外のファイルを変更しない約束を優先し、作業ツリーは変更していない。

適用すべき最小変更は、§4 の表①〜④を次の表へ置き換えること。E1〜E3 の各行で、`agent_project/*` が提供する汎用フックと、sibling 部品を含む codd-gate 側の処理を分けた。未設定時にパッケージが自動配線しない点も各行で判別できる。

```markdown
| 差し込み点 | `agent_project/*` の責務（汎用フック） | codd-gate 側の責務（任意 sibling を含む） | 未設定時 |
|---|---|---|---|
| E1 タスクの `- verify:`／charter `## acceptance` | 人が設定したコマンドを task verify または project evaluate で実行し、exit 0 を done／受入の根拠にする。コマンドの生成元や内容は判定しない | 人が `codd-gate check …` または `codd-gate verify --debt --max-broken 0 …` を記述する。`codd_gate_*.py` は E1 の値を自動生成・注入しない | 値がなければ通常の未定義時の扱い。codd-gate は動かない |
| E2 `regression_cmd`（設定／CLI） | 設定されたコマンドを毎タスクの verify PASS 後、done 確定前に実行する。exit 非 0 なら done にせず人へ回す。codd-gate の検出やコマンド生成はしない | 人または install 手順が明示実行した sibling `codd_gate_regression.py` だけが、使用可否を判定して `codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json` を yaml へ冪等 upsert する。実際の横断検査は codd-gate CLI が担う | `regression_cmd` は空のまま。横断検査を追加しない |
| E3 `intake_cmd`（設定／CLI） | 設定された単発コマンドを S0／watch idle の周期で実行し、stdout の task spec JSON を公開スキーマに従って取り込む。供給元を名指しせず、`id` で冪等化する | 人が `codd-gate tasks --debt [--cohort]` を設定する。codd-gate CLI が所見を task spec JSON に変換する。sibling `codd_gate_wiring.py` は推奨文字列の生成材料、`codd_gate_debt.py` は任意の正規化部品であり、パッケージから import しない | `intake_cmd` は空のまま。負債タスクを取り込まない |
```

表の直後にある `$KIRO_BASE_REV` と repos レジストリの説明は、そのまま残す。

## 検証

- `.codegraph/` がないことを確認し、通常の検索で正典 §4、§4.1 と `agent-project-design.md` §4.1 を照合した。
- E1 は exit code、E2 は `regression_cmd`、E3 は task spec JSON という汎用契約から外れていない。
- §4.1 の「有効化は yaml／CLI」「永続化は明示実行した `codd_gate_regression.py` のみ」「パッケージから sibling を import しない」と矛盾しない。
- `git status --short` は空で、許可範囲外を含め作業ツリーに変更がない。

## 前提・未解決事項・範囲外

- 前提: 「変更してよいのは `tools/agent-project` 配下のみ」を、対象文書の編集指示より強い書き込み制約として扱った。
- 未解決: 上記表を正典へ反映するには、`docs/designs/codd-gate-design.md` の編集権限を持つタスクが必要。
- 範囲外で確認した現行実装には、`agent_project/configfile.py` の起動時自動配線、`agent_project/doctor.py` と `agent_project/model.py` の sibling import が残る。今回は実装・テストを変更していない。

@followup `docs/designs/codd-gate-design.md` の編集を許可した文書タスクで、§4 の E1〜E3 表を上記へ置き換える。

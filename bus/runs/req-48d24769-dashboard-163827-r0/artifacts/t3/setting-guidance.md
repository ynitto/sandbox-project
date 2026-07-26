# 一貫性ゲート未結線時の正式な設定編集導線

## 成果／サマリー

正本は `tools/agent-project/README.md` の「一貫性ゲート（codd-gate 連携・オプション）」節（272–292 行）と判断した。dashboard が未結線時に示す正式な導線と文言は次のとおり。

1. 設定ファイルが見つかる場合
   - 検出した設定ファイルの実パスを示し、「設定ファイルを開く」で OS の既定エディタを開く。
   - UI 自身は設定を書き換えない。
   - 未結線のキーだけ、次の正準行を提示する（`<root>` は既知なら実際のプロジェクトルートで置換する）。

```yaml
regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
```

2. 設定ファイル自体が見つからない場合
   - README と同じ新ホームを使い、次の文言を示す。
   - 「agent-project の設定ファイルが見つかりません。ワークスペース直下に `.agents/agent-project.yaml` を作り、次の行を書く」
   - 設定ファイルが存在しない状態では、設定を開くボタンも sibling CLI も示さない。CLI の `--config` は既存ファイル必須であり、存在しない場合は終了コード 1 で止まり、半端な yaml を作らないため。

3. `regression_cmd` が未結線で既存設定ファイルがある場合
   - 手書きの代替として、`tools/agent-project/` で次を実行する導線を示せる。

```sh
python3 codd_gate_regression.py --config <検出した設定ファイルの実パス>
```

   - 補助文言: 「codd-gate を実測してこの 1 キーだけを冪等 upsert する。`--dry-run` なら書かずに結果だけ出す。codd-gate が未検出・バージョン/schema 非互換なら何も書かない」
   - CLI は `regression_cmd` のみを扱う。`intake_cmd` が未結線なら「`intake_cmd` に対応する注入 CLI は無いので、こちらは yaml を直接編集する」と明記する。

4. 状態表示に使う正式な区別
   - 両方結線済み: 見出し「有効」、各行「結線済み」。有効化導線は出さない。
   - 両方未結線: 見出し「未結線」。「まだ有効になっていないため、検査は行われません」。
   - 片方のみ: 見出し「一部のみ」。「未結線の項目は検査されません」。
   - `regression_cmd` / `intake_cmd` に別コマンドが入っている場合、その値は隠さず「別のコマンドが設定されています。一貫性ゲートの検査ではありません」と示す。
   - `regression_cmd` は `codd-gate … verify … --base`、`intake_cmd` は `codd-gate … tasks … --debt` の語順を満たすときだけ codd-gate 結線済みと扱う。単なるキーの存在や `make -s smoke` 等は結線済みにしない。

## 根拠

- `tools/agent-project/README.md:272-292`: 正準 2 行、新ホーム `.agents/agent-project.yaml`、sibling CLI、`--dry-run`、既存ファイル必須、`intake_cmd` の注入 CLI が無いことを規定。
- `tools/agent-project/codd_gate_regression.py`: README と同じ `regression_cmd` を生成し、既存設定へ 1 キーだけ冪等 upsert。未検出・非互換時は無変更。
- `tools/agent-project/tests/test_codd_gate_regression.py`: 正準値、root からの repos 推定、冪等性、dry-run、既存ファイル必須、未検出時 no-op、終了コードを固定。
- `tools/agent-dashboard/test/consistency-gate-ui.test.js`: README の正準 2 行との一致、実設定パス、設定不在時の `.agents`、CLI の表示条件、`intake_cmd` 直接編集、3 状態の表示を固定。
- `tools/agent-dashboard/test/consistency-gate.test.js`: 汎用フックに別コマンドがある場合を codd-gate 結線済みと誤判定しないことを固定。

## 検証内容と結果

- `node tools/agent-dashboard/test/consistency-gate-ui.test.js`: PASS (`consistency-gate-ui: ok`)
- `node tools/agent-dashboard/test/consistency-gate.test.js`: PASS (6 passed)
- `node tools/agent-dashboard/test/needs-gate-integration.test.js`: PASS (8 passed)
- `python -m unittest tools.agent-project.tests.test_codd_gate_regression`: PASS (34 tests)
- 対象 worktree の変更は行っていない。調査タスクのため `tools/agent-dashboard` 配下への書き込みは不要と判断した。

## 採用した前提・未解決事項・範囲外

- 「正式」は README を正本とし、設定の既定作成先は `.agents/agent-project.yaml` とした。既存設定が `.agent/`、ワークスペース直下、`.yml`、`.json` にある場合は、dashboard が検出した実パスを優先して案内する。
- `tools/agent-project/GUIDE.md:129-130,194` と `tests/test_codd_gate_regression.py` の一部説明・fixture には旧 `.agent/agent-project.yaml` 表記が残る。一方、README、現行の agent home 定義、dashboard テストは `.agents` を正準としている。動作コードは新旧双方を読み取るため、この表記差は本タスクでは修正しない。
- sibling CLI はソースツリーの `tools/agent-project/` で実行する前提。インストール版では単独スクリプトとして配置されないため、実行場所を省いた案内は不成立。
- agent-project 本体のフック実装、設定方式追加、UI からの状態書換は範囲外であり、変更していない。

@followup GUIDE と `test_codd_gate_regression.py` の旧 `.agent` 表記を `.agents` へ同期する文書整合タスクを別途検討する。

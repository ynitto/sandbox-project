# t4 一貫性ゲート有効化導線の契約確認

## 成果

完了条件を「dashboard README の有効化例を、agent-project README、`codd_gate_regression.py --help`、codd-gate の実 CLI parser と照合し、未結線時に表示してよい導線を確定すること」とした。リポジトリ内の変更は不要だった。

未結線時に表示してよい正式な設定値は次の 2 行。

```yaml
regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
```

- `<root>` は対象プロジェクトのルートへ人が置換する。
- YAML は表示した設定ファイルの該当トップレベルキーを追加・置換する。JSON は既存トップレベル object の同名プロパティだけを追加・置換し、ファイル全体を置換しない。
- 既存の別コマンドがある場合、置換するとその処理を失うため警告する。両方必要なら利用者が一つのコマンドへ合成する。
- dashboard は設定ファイルをエディタで開くところまでとし、設定更新、コマンド実行、タスク状態・done の書換えはしない。

既存 sibling CLI の正式な起動形は、リポジトリルートから次のとおり。

```bash
python3 tools/agent-project/codd_gate_regression.py \
  --config <状態 clone>/agent-project.yaml
```

`tools/agent-project/` を現在ディレクトリにする場合の `python3 codd_gate_regression.py --config ...` も同じ入口である。この CLI は既存 YAML ファイルだけを対象に `regression_cmd` 一つを冪等更新する。`--dry-run` は無変更確認、`--repos` と `--base` は生成値の上書きに使える。codd-gate が未検出・非互換なら更新しない。`intake_cmd` の注入 CLI はなく、設定を直接編集する。

## 検証

- dashboard README の 2 行と agent-project README の 2 行が文字列単位で一致することを確認。
- `codd-gate verify --help` で `verify`、`--base`、`--repos` を確認。
- `codd-gate tasks --help` で `tasks`、`--debt`、`--repos` を確認。
- `codd_gate_regression.py --help` で `--config`、`--repos`、`--base`、`--dry-run` と「`regression_cmd` のみ更新」を確認。
- `node tools/agent-dashboard/test/consistency-gate-ui.test.js`: PASS（README と表示例・CLI 名の同期を含む）。
- dashboard 全体の `npm test`: FAIL。依存 `yaml` が未導入で `feature-split.test.js` のロード時に停止した。コード失敗ではなく環境不足で、依存導入は調査タスクの範囲外とした。
- `git status --short`: 出力なし。`tools/agent-dashboard` 以下を含めリポジトリ変更なし。

## 前提・未解決・範囲外

- 「正式」は sibling 実装の argparse 契約と agent-project README を正とした。dashboard が自動探索した設定パスは、本体が明示 `--config` で使う実効パスとは限らないため、一致を断定しない。
- dashboard README/UI の CLI 例は `/path/to/.agents/agent-project.yaml` だが、現行 agent-project README の共有プロジェクト設定は `<状態 clone>/agent-project.yaml` が正式な置き場所。この差異は README 本文変更が out_of_scope のため未修正。
- 新規 CLI、設定自動更新、agent-project フック、UI からの状態書換えは実施していない。

@followup dashboard README/UI の CLI プレースホルダを `<状態 clone>/agent-project.yaml` に揃える別タスクを検討する。

# 一貫性ゲート未結線時の既存導線・確定文言

## 完了条件と採用した前提

- 完了条件は、`tools/agent-dashboard/README.md`、正本の `tools/agent-project/README.md`、sibling CLI 実装、既存 dashboard 表示を突き合わせ、未結線時に再利用すべき導線と文言を確定すること。
- このタスクは調査・仕様確定のみと解釈した。既存実装が条件を満たすため、リポジトリは変更しない。
- 「sibling CLI」は有効化を書き込む `tools/agent-project/codd_gate_regression.py` を指す。`codd_gate_wiring.py` は読み取り専用の診断 CLI であり、有効化導線には使わない。

## 確定した導線と文言

概要の「一貫性ゲート」で `regression_cmd` と `intake_cmd` を別々に表示し、各行に `結線済み` / `未結線` と `設定: あり` / `設定: なし` を出す。両方未結線なら見出しは `未結線`、片方だけなら `一部結線`、両方なら `結線済み` とする。別の汎用フックが設定済みなら値を隠さず、次を出す。

> 別のコマンドが設定されています。一貫性ゲートの検査ではありません。

未結線のキーだけ、設定ファイルへ追加する行を提示する。

```yaml
regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
```

続けて次を出す。

> `<root>` は対象プロジェクトのルートパスへ置き換えてください。

- 自動検出済み YAML がある場合: `設定ファイル <path> へ次の行を書く:` と `自動検出した設定ファイルを開く` ボタンを出す。ボタンは OS のエディタで開くだけで、書き換えない。
- 設定が無い場合: `agent-project の設定ファイルが見つかりません。ワークスペース直下に .agents/agent-project.yaml を作り、次の行を書く:` とする。CLI と開くボタンは出さない。
- JSON の場合: `既存トップレベル object` に未結線プロパティだけを追加または置換し、`ファイル全体は置き換えない` と明記する。YAML 専用 CLI は出さない。
- 別コマンドが設定済みの場合: 置換すると現在の処理を失う旨を警告し、残すか置き換えるかを人に決めてもらう。

既存 YAML があり、`regression_cmd` が未設定かつ未結線の場合だけ、手書きの代替として次の sibling CLI を提示する。

```bash
# リポジトリルートから
python3 tools/agent-project/codd_gate_regression.py \
  --config <状態 clone>/agent-project.yaml

# 同値: tools/agent-project/ へ移動して
python3 codd_gate_regression.py --config /path/to/.agents/agent-project.yaml
```

補足文言は次で確定する。

> codd-gate を実測してこの 1 キーだけを冪等 upsert する。`--dry-run` なら書かずに結果だけ出す。codd-gate が未検出・バージョン/schema 非互換なら何も書かない。

> `intake_cmd` に対応する注入 CLI は無いので、こちらは yaml を直接編集する。

`codd_gate_regression.py` は既存 YAML を必須とし、`regression_cmd` だけを書き込む。成功時 JSON に `intake_cmd` の推奨値も出すが、それは書き込まない。

## 検証

- `node tools/agent-dashboard/test/consistency-gate.test.js`: 15 passed。
- `node tools/agent-dashboard/test/consistency-gate-ui.test.js`: pass。
- `node tools/agent-dashboard/test/needs-gate-integration.test.js`: 8 passed。
- 対象の main / renderer / 2 テストに ESLint を実行し、終了コード 0。
- `git status --short`: clean。`tools/agent-dashboard` を含めリポジトリ変更なし。

## 未解決事項・範囲外

- dashboard の自動探索設定と、agent-project が `--config` で実際に使う設定の一致は status 契約から確認できないため、画面も断定しない。
- dashboard は codd-gate の実在・互換性・実行成功を検査せず、設定文字列だけで結線状態を表示する。
- agent-project 本体のフック実装、設定の自動書換、UI からの状態・done 書換は範囲外であり未変更。

# configfile.py / sibling 自動配線パターン調査

## (a) 成果・サマリー

結論: `agent_project` 本体を特定ゲート非依存に保つための差し込み点は既に二層ある。

1. **診断プロバイダの汎用フック**
   - `agent_project/hooks.py:15-18` の `HOOK_CAPABILITIES` が本体側の契約:
     - `wiring.detect` → `detect_wiring`
     - `wiring.findings` → `doctor_findings`
   - `agent_project/hooks.py:44-56,110-132` の `_hook_provider` が唯一の解決口。
     解決順は `hooks:` の明示指定（フル能力キー優先、次に `wiring` の系統キー）→ sibling の能力スキャン。
     明示名の解決失敗時は別 provider へ自動 fallback しない。
   - sibling スキャンは `agent_project/hooks.py:74-98`。`tools/agent-project/*.py` を昇順走査し、
     ソースに `def <契約名>(` がある module だけを import する。
   - `agent_project/doctor.py:343-364` が provider を能力名で取得し、検出結果の型を本体で解釈せず
     `detect.detect_wiring(...)` → `render.doctor_findings(...)` と中継する。
     `cmd_doctor` への合流点は `doctor.py:571-573`。

2. **実処理の汎用コマンドフック**
   - `configfile.py:143-147` の既定値と `configfile.py:271-287` の通常の設定解決を通じ、
     `regression_cmd` / `intake_cmd` / `hooks` を受ける。
   - `configfile.py:429-434` は値を `Config` へそのまま渡すだけで、バイナリ検出や実行時補完をしない。
   - `regression_cmd` の実行点は `agent_project/mr.py:576-587`。各タスク verify 成功後、done 確定前に
     `cfg.workdir` でグローバル回帰検査として実行する。
   - `intake_cmd` の実行点は `agent_project/model.py:540-568`。任意検出器の JSON stdout を汎用 intake として取り込む。
     通常パス開始時は `mr.py:679`、watch idle 中は `loop.py:677` から呼ばれる。

### codd_gate sibling を利用する既存経路

- **診断を明示配線する経路**

  ```yaml
  hooks:
    wiring: codd_gate_wiring
  ```

  `agent-project.yaml.example:182-189` に既存例がある。
  `codd_gate_wiring.py` は `detect_wiring = probe_wiring`、
  `doctor_findings = render_findings` の alias で契約を満たす。
  alias は `def detect_wiring(` ではないため、意図的に sibling 自動スキャンには当選せず、
  `hooks:` を書いた場合だけ doctor に結線される。

- **実処理を明示設定する経路**

  ```yaml
  regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
  intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
  ```

  本体にとっては任意の shell command であり、codd-gate 固有知識は不要。

- **設定ファイルへ恒久注入する既存 CLI**

  `python3 codd_gate_regression.py --config <agent-project.yaml>`

  `codd_gate_regression.py:203-243` が検出→推奨 `regression_cmd` 生成→既存 YAML への冪等 upsert を行う。
  これは実行時自動配線ではなく、明示的に設定ファイルを書き換える経路。
  `intake_cmd` は注入しないため、必要なら上記 YAML を明示する。

- **書き込まず診断・推奨値を得る既存 CLI**

  `python3 codd_gate_wiring.py --config <agent-project.yaml>`

  `codd_gate_wiring.py:250-277` が現在の両コマンドを読み、利用可否・結線状況・推奨値を JSON 出力する。

### 後続実装で使うべき差し込み点

- provider 固有名を本体へ戻さず、診断能力の追加・交換は `HOOK_CAPABILITIES` と `_hook_provider` 境界で行う。
- gate の実行は新しい runtime hook を増やさず、既存 `regression_cmd` / `intake_cmd` を使う。
- codd-gate の有効化を自動化する必要がある場合も、`configfile.build_config` のメモリ補完ではなく
  sibling CLI による明示・恒久設定を使う。
- 現状の要求には新規コード変更は不要。旧 `_apply_codd_gate_auto_wiring`、
  `_codd_gate_wiring_module`、`doctor_codd_gate_findings` は現物に存在しない。

## (b) 検証内容と結果

- `.codegraph/` は worktree に存在しなかったため、CodeGraph は使わず `rg` とソース読解で呼出経路を確認。
- `python3 -m unittest tools/agent-project/tests/test_codd_gate_wiring.py tools/agent-project/tests/test_codd_gate_regression.py`
  - **63 tests / OK**
- `python3 -m unittest tools/agent-project/tests/test_agent_project.py -k CoddGateNoAutoWiring`
  - **4 tests / OK**
  - repos.json が存在しても自動補完しないこと、明示コマンドを改変しないこと、旧関数が無いことを確認。
- `rg -n "codd_gate" agent_project/configfile.py agent_project/doctor.py agent_project/hooks.py`
  - **一致なし**。本体三ファイルに provider 固有名が無い。
- 作業リポジトリのコードは変更していない。成果物は本調査報告のみ。

## (c) 採用した前提・未解決事項・範囲外

### 採用した前提

- 本タスクの完了条件は、後続実装が利用できる既存パターン、設定経路、実行点、制約を根拠付きで特定すること。
  調査結果だけで判断可能なため、コード変更は不要とした。
- 「自動配線」は二つを区別した:
  1. provider の能力ベース解決（`hooks.py`）
  2. `regression_cmd` / `intake_cmd` の値を実行時に勝手に生成する挙動
  後者は設計上禁止され、前者も `codd_gate_wiring` については明示設定時のみ有効。

### 未解決事項

- `configfile._normalize_hooks` は非 mapping を `{}` に変換してから `Config` に渡す。
  一方 `doctor._hook_misconfig_findings` は非 mapping を warn する設計なので、通常の設定ロード経路では
  元の型情報が失われ、その warn 分岐へ到達できない。今回の調査範囲では修正していない。
- `_HOOK_CACHE` は capability だけをキーにするプロセス全体キャッシュ。
  同一プロセス内で異なる `cfg.hooks` を順次使う場合、先に解決した provider/None が残る。
  現行テストは cfg 切替時に明示的に cache clear している。複数 project が異なる provider を使う要件があるなら
  cache key またはライフサイクルの見直しが必要。
- コメント上は sibling 自動検出を既定としているが、現在の sibling 群は契約名を `def` で公開しないため全件空振りする。
  これは codd-gate の零設定再結線を防ぐ既存テストで意図的に固定されている。

### 範囲外で見つけた問題

`@followup configfile の hooks 正規化で失われる型エラーを doctor へ伝える方法を検討する。`

`@followup 複数 project / 複数 provider を同一プロセスで扱う場合の _HOOK_CACHE 汚染を回帰テスト化する。`

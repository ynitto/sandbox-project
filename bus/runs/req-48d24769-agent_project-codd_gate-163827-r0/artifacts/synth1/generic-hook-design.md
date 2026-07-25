# agent-project 汎用フック確定仕様

## 結論

`agent_project` 本体の確定契約は、診断用の capability provider と実処理用の command hook の二層とする。新しい抽象化や provider 固有の自動配線は追加しない。

| 用途 | 設定・能力名 | provider の必須 callable |
|---|---|---|
| 配線状態の検出 | `hooks.wiring.detect`（短縮指定は `hooks.wiring`） | `detect_wiring` |
| doctor 所見への変換 | `hooks.wiring.findings`（短縮指定は `hooks.wiring`） | `doctor_findings` |
| done 前の全体回帰 | `regression_cmd` | shell command |
| 外部検出結果の intake | `intake_cmd` | JSON object/array を stdout へ出す単発 shell command |

内部名は既存の `HOOK_CAPABILITIES`、`_hook_provider`、`_hook_resolution_error`、`_normalize_hooks`、`_hook_misconfig_findings`、`doctor_wiring_findings` を維持する。旧 provider 固有名を別の private 名へ一対一改名しない。

## sibling 探索規則

1. 探索起点は `Path(__file__).resolve().parent.parent`、すなわち `agent_project/` の一階層上である。
2. 直下の `*.py` だけをファイル名昇順で走査する。再帰探索はしない。
3. stem が `_` で始まるファイル、Python identifier でないファイルは除外する。
4. capability に必要な callable ごとに、ソース中の行頭 `def <契約名>(` を前置フィルタとする。alias、代入、動的属性だけの module は自動検出しない。
5. 前置フィルタ通過後に sibling を `sys.path` へ加え、stem を module 名として import する。
6. import 例外、必須属性欠落は候補不成立として次へ進み、全必須属性を持つ最初の module を採用する。
7. 候補不在は `None` とし、呼び出し側は no-op へ縮退する。解決結果は capability 単位で `None` も含めてキャッシュする。

この規則により、契約名を alias で公開する provider は明示設定時だけ結線される。零設定での意図しない再結線を防ぐ現行動作を維持する。

## 明示設定の優先順位

### provider 解決

高い順に次のとおり。

1. capability 完全キー: `hooks["wiring.detect"]` / `hooks["wiring.findings"]`
2. capability 系統キー: `hooks["wiring"]`
3. `hooks` 未指定時の sibling 能力探索
4. provider 不在として `None`

完全キーまたは系統キーが空でない module 名を指定して解決に失敗した場合、sibling 探索へ fallback しない。人の明示意図を別 provider で黙って置換せず、doctor の config/warn で知らせる。

### 設定値の供給

`resolve_config` の共通規則どおり、CLI namespace の非 `None` 値が設定ファイルより優先し、設定ファイルが組み込み既定より優先する。`hooks` には専用 CLI option がないため通常は設定ファイル値が使われ、既定は `{}` である。`regression_cmd` / `intake_cmd` は CLI 明示値、設定ファイル値、`None` の順となる。`repos.json` の存在や provider 検出結果で両 command を補完・上書きしない。

## ファイル別変更表

### 必須変更

| ファイル | 確定変更 | 根拠 |
|---|---|---|
| `agent_project/charter.py` | `export_repo_registry` の docstring から特定 provider 名2件を除き、「charter を直接読まない外部ツールへ渡す派生物」と表現する。 | registry 出力は provider 非依存。 |
| `agent_project/configfile.py` | `intake_cmd` の既定値コメントと CLI help の具体例から特定 provider 名を除き、JSON を返す外部検出器という契約だけを残す。 | 設定値は任意 command を素通しする。 |
| `agent_project/model.py` | `_parse_intake_records` と `run_intake` の docstring から特定 provider 名を除き、外部の決定的検出器という表現へ統合する。 | parser/runtime は JSON 契約だけに依存する。 |
| `agent_project/mr.py` | regression の cwd コメントを「グローバル検査が root の registry と基準 revision を解決できない」に一般化する。動作は変えない。 | `cfg.workdir` 実行が契約で、provider 名は理由の本質でない。 |
| `agent_project/verify.py` | 無出力 command のコメントを一般化し、`_KNOWN_COMMAND_WORDS` から特定 CLI 名を削除する。 | ハイフン付き CLI は既存の汎用 regex で認識されるため専用 allowlist は不要。 |
| `tests/test_config.py` | `TestCoddGateNoAutoWiring` と旧 private symbol 不在テストを汎用名・観測可能な契約へ置換する。`repos.json` で command を補完しない3ケースは維持し、置換テストは `hooks` 設定が `Config.hooks` へ渡ることを確認する。 | private 名の grep をテスト契約にせず、「本体は差し込み点だけ」を振る舞いで固定する。 |
| `tests/test_doctor.py` | fake provider と `_hook_provider` の capability 別差し替えで `doctor_wiring_findings` の統合テストを追加する。引数転送、片側欠落、provider 例外、明示解決失敗 warning を最小ケースで固定する。 | 現行241件では本体 doctor と generic provider の境界が未検証。 |

### 変更しない

| ファイル | 理由 |
|---|---|
| `agent_project/hooks.py` | 汎用名、探索規則、優先順位がすでに確定仕様を満たす。 |
| `agent_project/doctor.py` | provider 固有名を持たず、capability と opaque judgment だけを中継する。 |
| `agent_project/loop.py` | intake の watch idle 呼出しは既存契約どおり。 |
| `tests/test_backlog.py` | intake の object/array、冪等、interval、異常縮退、watch 再開を既に検証する。 |
| `tests/test_autonomy.py` | regression の task verify 後実行、pass/fail、`cfg.workdir` を既に検証する。 |
| `tests/test_codd_gate_*.py`、`codd_gate_*.py`、`agent-project.yaml.example` | `agent_project` 本体外の provider 実装・固有テスト・明示設定例であり、分離境界の外側として残す。 |

## 受入条件

```sh
! rg -n -i 'codd[_-]gate' tools/agent-project/agent_project
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
(cd tools/agent-project && python3 -m unittest \
  tests.test_backlog tests.test_config tests.test_doctor tests.test_autonomy \
  tests.test_codd_gate_wiring tests.test_codd_gate_regression)
python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py'
```

基準実績は関連241件成功、provider 境界111件成功。t2 の `tests/test_agent_project.py` 実行記録は現行 HEAD では再現不能なので、`tests.test_config.TestCoddGateNoAutoWiring` の4件成功を正しい旧証跡として扱う。

## 矛盾・欠落の統合

- t1 の「固有識別子0件」と残存文字列8件は矛盾しない。前者は import/定義/参照、後者は説明・例示と command 語彙である。最終受入はより強い「本体全文0件」を採用する。
- t2 の削除済み `tests/test_agent_project.py` を使った証跡は gate1 の再検証結果で置換した。
- t3 の provider テスト列挙漏れは、`test_codd_gate_*.py` 全件実行を受入条件にして補完した。
- `_normalize_hooks` が不正型を `{}` に変えるため、通常ロード後は doctor の「hooks 型不正」分岐へ届かない問題は本変更の命名・分離に不要なので変更しない。
- capability だけをキーにする `_HOOK_CACHE` の複数 project 間汚染は、現行の一 project 一 process 前提では本変更の必須条件にしない。

@followup 同一プロセスで異なる project/provider を切り替える要件が生じたら、`_HOOK_CACHE` のキーへ設定 identity を含める。
@followup hooks 不正型を doctor で通知する要件があるなら、正規化前の raw 値または validation error を `Config` へ保持する。

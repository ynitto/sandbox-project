# codd-gate 連携境界の抽出

## 前提と完了条件

- 対象は `docs/designs/codd-gate-design.md` の §4 と §4.1。差し込み点 E1〜E3、自動検出を構成する層、`agent_project` と sibling 部品の責務を、後続作業が引用できる行番号付きでまとめる。
- 「自動検出レイヤ」は単一モジュール名ではなく、§4.1 が分ける「実在するか」「使ってよいか」「実引数はどう組むか」と、その実測を束ねる部品群を指すと解釈した。
- 調査タスクなのでリポジトリは変更しない。正典と現物コードの不一致は修正せず、未解決事項として記録する。

## 差し込み点 E1〜E3

正典は、codd-gate が agent-project の公式フック E1〜E3だけを使い、外せば元へ戻るオプション連携だと定める（`docs/designs/codd-gate-design.md:239-244`）。各点の契約は次のとおり。

| 差し込み点 | codd-gate の用途 | 既存フックの契約と責務 | 根拠 |
|---|---|---|---|
| E1 `verify` / charter `acceptance` | `verify --debt` でプロジェクト受入を判定し、タスクの `verify` では `check` を done の根拠にする | タスク固有または上位 evaluate の完了条件。exit 0 が PASS | `docs/designs/codd-gate-design.md:264,266,331-341`; `docs/designs/agent-project-design.md:217,224` |
| E2 `regression_cmd` | `verify --base "$KIRO_BASE_REV"` を毎タスクの verify PASS 後、done 確定前に実行する | 全タスク共通の横断検査。失敗時は done にせず人へ回す。「止める」方向だけに働く | `docs/designs/codd-gate-design.md:263,269-271,331-334`; `docs/designs/agent-project-design.md:218,224` |
| E3 `intake_cmd` | `tasks --debt [--cohort]` を watch 周期で実行し、負債を修復タスクとして取り込む | 周期 pull のタスク供給。stdout は enqueue JSON、`id` は冪等キー。単発・有界で、失敗はループを止めない | `docs/designs/codd-gate-design.md:265,335-338`; `docs/designs/agent-project-design.md:219,224` |

補助契約として、agent-project は charter から `<root>/repos.json` を生成し、codd-gate は `--repos` で読むだけで charter を読まない（`docs/designs/codd-gate-design.md:267`）。タスク入力契約の所有者は agent-project、供給側は共通 task schema への変換だけを担う（`docs/designs/agent-project-design.md:227-232`）。repos は独立スキーマで、外部ツールへのファイル境界になる（同 `:234-244`）。

## 自動検出レイヤと sibling 部品

§4.1 は sibling 群を、手書きするフック値の生成・判定・永続化を助ける任意部品と位置づける。パッケージ起動時に値を差し込む層ではない（`docs/designs/codd-gate-design.md:273-280`）。責務の分割は次のとおり。

| 部品 | 既存責務 | 根拠 |
|---|---|---|
| `codd_gate_detect.py` | CLI 実体の解決と、バージョン、repos schema、capability の生の検出値。利用可否は判断しない | 設計 `:284`; 実装 docstring `tools/agent-project/codd_gate_detect.py:2-21` |
| `codd_gate_status.py` | 生の検出結果を `CoddGateStatus` にまとめ、未検出・非互換を `usable=False` / `command() is None` へ縮退させる | 設計 `:285,296-299`; 実装 docstring `tools/agent-project/codd_gate_status.py:2-20` |
| `codd_gate_routing.py` | regression、intake、acceptance 共通の `--repos` と `--repo-dir` 実引数を組み立てる | 設計 `:286` |
| `codd_gate_base.py` | `$KIRO_BASE_REV`、charter の `base`、`HEAD~1` の順で差分 base を解決する | 設計 `:287` |
| `codd_gate_debt.py` | intake の JSON を task schema 相当に正規化し、不正レコードだけを隔離する任意パーサ | 設計 `:288,292-295` |
| `codd_gate_wiring.py` | 生の検出を短絡順で実測し、手書き値の結線有無を判定し、推奨コマンド文字列と doctor 所見を返す。設定への書き込みはしない | 設計 `:289`; 実装 docstring `tools/agent-project/codd_gate_wiring.py:2-24` |
| `codd_gate_regression.py` | 使用可能なときだけ `regression_cmd` を生成し、明示実行された CLI として yaml へ最小差分で冪等 upsert する | 設計 `:290,306-314`; 実装 docstring `tools/agent-project/codd_gate_regression.py:2-27` |

検出レイヤの流れは `detect`（実測値）→ `status`（使用可否と no-op 縮退）→ `routing` / `base`（実引数）→ `wiring`（実測の束ね、結線判定、推奨値）→ `regression`（人が明示実行する永続化）となる。`debt` は E3 出力の任意パーサで、検出経路とは別枝にある。

## `agent_project` と sibling の境界

- `agent_project/*` の責務は汎用フック E1〜E6 の提供だけ。codd-gate を名指し、import、自動配線せず、フック値も書かない（`docs/designs/codd-gate-design.md:248-251`）。
- `codd_gate_*.py` は `tools/agent-project/` 直下のトップレベル sibling。標準ライブラリだけで動き、Config / Charter / Task に依存せず、パッケージから import されない。全削除してもパッケージの挙動は同じでなければならない（同 `:252-255`）。
- フック値の有効化は人または install 手順による yaml / CLI 設定、永続化は明示実行された `codd_gate_regression.py` だけ。`build_config` による起動時注入は禁止される（同 `:256-259,301-314`）。
- `.agent/agent-project.yaml` は人専有ファイルであり、自動 Config 生成を採らない理由にもなっている（同 `:316-320`）。

## 検証結果

- `docs/designs/codd-gate-design.md:239-352` と、参照先のフック正典 `docs/designs/agent-project-design.md:208-252` を行番号付きで照合した。
- sibling 7ファイルの存在を確認し、`detect`、`status`、`wiring`、`regression` は実装 docstring でも責務を照合した。
- `git status --short` は空で、指定 worktree に変更がないことを確認した。

## 未解決事項と範囲外の問題

現物コードは上記の正典と一致しない。

- `agent_project/configfile.py:201-229` の `_apply_codd_gate_auto_wiring()` が検出結果から `cfg.regression_cmd` と `cfg.intake_cmd` をメモリ上で補い、`build_config` が同関数を呼ぶ（同 `:376`）。これは「起動時の Config 生成が値を差し込まない」という `codd-gate-design.md:256-259,301-304` に反する。
- `agent_project/model.py:494-515,527-530,552-557` は `codd_gate_debt` を遅延 import し、intake の通常経路で使う。
- `agent_project/doctor.py:287-320` は `codd_gate_wiring` を遅延 import し、doctor 所見へ組み込む。

したがって「パッケージから sibling を import しない」「sibling を丸ごと削除しても同一挙動」という記述は現状コードには成立しない。agent-project / dashboard の実装変更とテスト改修は明示されたスコープ外なので、この成果では修正していない。

@followup agent-project 実装から `_apply_codd_gate_auto_wiring` と sibling import 経路を除き、汎用フックだけを残す整理を別タスクで行う。受入には §4.2 の grep に加え、`agent_project/` 内の `codd_gate` 名指し・import がゼロであることを検査する。

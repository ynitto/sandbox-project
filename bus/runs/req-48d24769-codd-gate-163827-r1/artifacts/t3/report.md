# t3 成果報告

## サマリー

`docs/designs/codd-gate-design.md` §4 と §4.1 から、E1〜E3 の境界を抽出した。設計書が固定している境界は次のとおり。

- `agent_project/*` は外部 CLI を受ける汎用フックだけを持つ。
- codd-gate のコマンド文字列は、人か install 手順が YAML/CLI に書く。
- `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品であり、パッケージは import しない。
- sibling 部品を削除してもパッケージの挙動は変わらない。

### E1: task verify / charter acceptance

| 区分 | 現行記述 |
|---|---|
| agent-project の汎用フック | タスクの `- verify:` は S3 の done 根拠、charter の `## acceptance` は上位 evaluate のプロジェクト受入根拠。workdir、`verify_cwd` または workspace clone で実行し、`$KIRO_BASE_REV` を受け取る。exit 0 が PASS。状態または差分を確認する有界コマンドであることを契約とする。 |
| codd-gate 固有処理 | 修復タスクの verify に `codd-gate check …` を置き、接続、参照解決、鮮度などの期待状態を判定する。charter acceptance には `codd-gate verify --debt --max-broken 0 …` のような負債ラチェットを置く。値は人か install 手順が書き、パッケージは生成も自動配線もしない。 |
| 任意 sibling 部品 | `codd_gate_routing.py` が repos と repo-dir の実引数を組み立てる。§4.1 に E1 専用の永続化処理はなく、acceptance の値は手書きまたは install 手順の担当。 |

### E2: regression_cmd

| 区分 | 現行記述 |
|---|---|
| agent-project の汎用フック | 設定または CLI の `regression_cmd` を、タスク verify PASS 後かつ done 確定前に実行する。E1 と同じ cwd/env を使い、exit が 0 でなければ done にせず人へ送る。全タスク共通、タスク非依存、有界で、「止める」方向だけに作用する。 |
| codd-gate 固有処理 | `codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json` を値として置き、毎タスクの差分整合性を横断検査する。NG なら done を止める。 |
| 任意 sibling 部品 | `codd_gate_detect.py`、`codd_gate_status.py`、`codd_gate_routing.py`、`codd_gate_base.py`、`codd_gate_wiring.py` が実体、版、schema、能力、実引数、base rev、推奨値を判定する。`codd_gate_regression.py` は人か install 手順が明示実行したときだけ推奨値を生成し、YAML の `regression_cmd` 1 行を冪等に永続化する。未検出または非互換なら書かない。 |

### E3: intake_cmd

| 区分 | 現行記述 |
|---|---|
| agent-project の汎用フック | 設定または CLI の `intake_cmd` を S0 と watch idle で実行する pull 型の供給口。cwd は workdir。stdout は `enqueue --json` と同じ task spec または配列で、`id` を冪等キーに backlog へ取り込む。コマンドは単発かつ有界。exit 非 0 や非 JSON は無視し、ループを止めない。 |
| codd-gate 固有処理 | `codd-gate tasks --debt [--cohort] --repos <root>/repos.json` を値として置き、負債を決定的な ID を持つ修復タスクへ変換する。周期実行は agent-project が担い、codd-gate 自体は 1 パスで終了する。手動供給は E4 の `enqueue --json` または `inbox/` を使う。 |
| 任意 sibling 部品 | `codd_gate_wiring.py` が推奨 `intake_cmd` を組み立てる。`codd_gate_debt.py` は object/array の stdout を task spec に正規化し、不正レコードだけを隔離する任意パーサ。ただし §4 の境界では、パッケージからこの sibling を import しない。`intake_cmd` の永続化は人か install 手順の担当で、`codd_gate_regression.py` の対象外。 |

## 現物コードとの突合

E1 の実行部は汎用のままだった。`agent_project/project.py:26` の `evaluate_acceptance()` は charter のコマンド列を実行し、`agent_project/mr.py:461` 以降は task verify を処理する。ここに codd-gate 専用分岐は見当たらない。

E2 と E3 は設計書の境界へまだ到達していない。現在のパッケージには次の codd-gate 固有結合が残る。

- `agent_project/configfile.py:201-229,376`: `_apply_codd_gate_auto_wiring()` が sibling の検出結果から、メモリ上の `cfg.regression_cmd` と `cfg.intake_cmd` を自動設定する。
- `agent_project/doctor.py:287-318,528`: `codd_gate_wiring` を遅延 import し、codd-gate 固有の doctor 所見を収集する。
- `agent_project/model.py:494-552`: `codd_gate_debt` を遅延 import し、E3 の stdout を codd-gate 固有パーサで正規化する。

したがって、設計書と README が完了条件にしている次の否定検索は現 HEAD で失敗する。

```bash
! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

これは今回の変更対象外なので修正していない。

## 検証内容と結果

- `.codegraph/` は対象 worktree に存在しなかったため、`rg` と `git grep` で参照箇所を確認した。
- `docs/designs/codd-gate-design.md` §4、§4.1 と `docs/designs/agent-project-design.md` §4.1 の E1〜E3 契約を突き合わせた。
- `tools/agent-project/codd_gate_*.py` の公開関数と、`agent_project/configfile.py`、`model.py`、`doctor.py`、`mr.py`、`project.py` の実行経路を確認した。
- 上記の否定検索は 14 行に一致し、exit 0 にならない状態であることを確認した。
- `git status --short --branch` は作業前に `## HEAD (no branch)`。リポジトリ配下のファイルは編集していない。

## 前提・未解決事項・範囲外

- 「現行記述」は、指定された設計正典の規範記述を指すものと解釈した。実装の現状は別枠で突合し、規範と混同しないようにした。
- t2 の報告どおり、この worktree の HEAD は `abe5906a18149a1ab4a4c97654bfa7d7950eb763` で、最新 main `1d6a516f2e40c42ad2a7f8de5f70981b5a140e39` へ rebase 済みではない。本報告は渡された HEAD の内容に基づく。
- agent-project と dashboard の実装変更、テスト改修は範囲外のため行っていない。
- 未解決事項は、E2/E3 のパッケージ内 codd-gate 固有結合を誰がどの履歴上で除去するか。履歴更新後に同じ否定検索を再実行する必要がある。

@followup E2/E3 の境界整理タスクで、`configfile.py` の自動配線、`doctor.py` の固有診断、`model.py` の固有パーサ結合をパッケージ外へ移し、README 記載の否定検索を受入条件にする。

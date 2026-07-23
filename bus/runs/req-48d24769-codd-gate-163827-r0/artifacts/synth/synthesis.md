# codd-gate 連携の正典境界

## 確定事項

1. `agent_project/*` は E1〜E6 の汎用フックだけを持つ。codd-gate を名指し、`codd_gate_*` を import、自動配線してはならない。未設定のフック値は空のまま扱う。
2. `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品とする。パッケージ型に依存せず、全削除しても `agent_project` の挙動は変わらない。
3. ディスクへの永続化は、人または install 手順が明示実行する `codd_gate_regression.py` だけが担う。対象は `regression_cmd` の yaml への冪等 upsert であり、起動時の Config 生成は書き込まない。
4. 連携の有効化経路は yaml または CLI だけとする。任意 sibling の検出は生成可否と推奨値を判断する材料であり、`build_config` に値を注入する配線ではない。

根拠は `docs/designs/codd-gate-design.md:239-259,273-327` と `docs/designs/agent-project-design.md:208-252`。E1 は task verify / charter acceptance、E2 は `regression_cmd`、E3 は `intake_cmd` であり、結合はコマンド文字列と `schemas/` のデータ契約に限る。

## 文書ごとの最小変更点

### `docs/designs/codd-gate-design.md`

§4 と §4.1 の規範本文は上記境界をすでに表しているため維持する。§4.2 の必須完了ゲートだけを、`_apply_codd_gate` 専用 grep から package 限定の厳密 grep へ変更する。

```sh
! git grep -nE '_codd_gate|import codd_gate' -- tools/agent-project/agent_project
```

現行の `! git grep _apply_codd_gate -- tools/agent-project` は `doctor.py` と `model.py` の sibling import を見逃すため、宣言した境界の受入にならない。厳密 grep を「任意例」ではなく唯一の必須ゲートとし、旧ゲートを正典から外す。検索範囲は sibling とそのテストを許容するため `agent_project/` に限定する。コメント中の一般的な `codd-gate` 言及まで禁じる総当たり grep は採らない。

### `tools/agent-project/README.md`

現在のブランチ差分を採用する。すなわち、起動時自動配線の説明を削り、package 非依存、sibling 任意、有効化は yaml/CLI、永続化は明示実行した `codd_gate_regression.py` という利用者向け要約に置き換える。追加の説明は不要。

### `docs/designs/agent-project-design.md`

変更不要。§4.1 は E1〜E6 を供給元非依存の外部 CLI フックとして定義済みで、E3/E4 の入力契約と repos schema の所有権も境界と一致する。

## 現物との差

文書を現行実装へ合わせる案は却下する。`agent_project/configfile.py:201-229,376` の起動時注入、`agent_project/doctor.py:287-320,528` の `codd_gate_wiring` import、`agent_project/model.py:494-515,552-557` の `codd_gate_debt` import は正典違反だが、実装変更とテスト改修は今回の範囲外である。

また、`codd_gate_regression.py:180-182` は現状 `detect_status()` しか使わず、§4.1 が求める version、schema、capability の完全な実測に達していない。正典の no-op 条件は目標契約として残し、実装を後続で `detect_wiring()` 相当へ合わせる。

t2 の README 抽出内容は採用するが、「未解決事項なし」は採用しない。README は目標境界を正しく記述している一方、現行 package とは不一致である。t3 の 72 件 PASS も現行挙動の回帰確認にすぎず、正典準拠の根拠には使わない。gate1 の再検証では関連 5 ファイル 81 件が PASS しており、件数差は境界判断に影響しない。

@followup agent-project 実装から起動時自動配線、doctor 固有検出、debt 固有 import を除去し、`codd_gate_regression.py` の生成可否判定を version/schema/capability の実測へ合わせる。関連テストは新しい正典境界を検査する期待値へ改修する。

# synth2 成果報告

## 成果

- `tools/agent-project/README.md` を gate2 に合わせて修正した。
- 責務境界を「`agent_project/*` は汎用フックのみ、`codd_gate_*.py` は任意 sibling 部品」に固定した。
- 永続化前の検出方式を現行実装どおり「codd-gate の実体検出のみ」と明記した。
- パッケージ境界の完了条件を、README 自身や任意 sibling 部品を誤検出しない次の一行へ統一した。

  ```sh
  ! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project
  ```

- 設計書は書込許可範囲外のため、同じ責務境界、コマンド名、検出方式、完了条件を
  [`design-proposed-edit.md`](design-proposed-edit.md) にまとめた。

## 矛盾の解消

t19 案の「`codd_gate_regression.py` がバージョン、schema、capability を検査する」は、
gate2 と現行コードに反するため採用しなかった。現行 CLI は `detect_status()` を呼び、実体だけを確認する。
完全検査は未実装の後続課題として分離した。

## 範囲

リポジトリ内の変更は `tools/agent-project/README.md` だけ。`agent_project/*`、dashboard、テスト、
書込範囲外の `docs/designs/codd-gate-design.md` は変更していない。

`git diff --check` は PASS。新しい完了ゲートは現行コードの
`configfile.py`、`doctor.py`、`model.py` に残る禁止語を検出し、想定どおり未完了を返した。

@followup `design-proposed-edit.md` を `docs/designs/codd-gate-design.md` §4〜§4.2 に反映する。

@followup `codd_gate_regression.py` の永続化前判定を `detect_wiring()` に統合し、
バージョン、repos schema 互換性、capability の不適合時に値を書かない受入を追加する。

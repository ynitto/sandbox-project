切り口: 禁止事項と許可された拡張点を一文で対にし、実装時に判定できる境界へ固定した。

## 成果

`tools/agent-project/README.md` の「一貫性ゲート（codd-gate 連携・オプション）」冒頭に、次の不変条件を明記した。

- `agent_project` パッケージが提供するのは汎用フックだけである。
- `agent_project` は `codd_gate_*` を import、直接結合、依存しない。
- codd-gate 連携は、共通スキーマと、人または install 手順が E1〜E3 の汎用フックへ設定するコマンド文字列に限る。

後段にあった同趣旨の文は重複を避けて短くした。変更は README のみで、実装とテストには触れていない。

## 検証

- `git diff --check`: PASS。空白エラーはない。
- README の対象範囲を再読し、「汎用フックだけ」「`codd_gate_*`」「import」「直接結合」「依存」が一続きの規範として記載されたことを確認した。
- 正典ゲート `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`: 現 HEAD では FAIL。`configfile.py`、`doctor.py`、`model.py` に計 14 件が残る。これは README が示す目標境界の未達を正しく検出している。
- 文書だけの変更のため、コードテストと型チェックは実行していない。

## 前提・未解決事項

- 完了条件は、目標境界を README の一貫性ゲート節に明示することと解釈した。現行実装を境界へ合わせる作業は元要求の範囲外である。
- `codd_gate_*.py` は `tools/agent-project/` 直下の任意 sibling 部品であり、パッケージから削除可能である、という依存成果の統一境界を採用した。
- 未解決: 正典ゲートが検出した 14 件の専用結合は残っている。今回の許可範囲に従い修正していない。

@followup 別タスクで `configfile.py`、`doctor.py`、`model.py` の専用結合と関連テストを汎用フック境界へ合わせる。

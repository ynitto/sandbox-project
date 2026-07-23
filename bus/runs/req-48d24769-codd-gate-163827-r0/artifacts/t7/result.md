# t7 成果報告

## 成果

リポジトリは未変更。`docs/designs/codd-gate-design.md` §4 と §4.1 に、要求された境界がすでに明記されていることを確認した。

- 有効化経路は、人または install 手順が `regression_cmd`、`intake_cmd`、acceptance の値を yaml か CLI に書く場合に限定されている。
- `agent_project/*` は汎用フックだけを提供し、codd-gate の名指し、import、自動配線を行わない方針になっている。
- `build_config` が値を差し込む経路を設けず、未設定なら連携なしで通過すると規定されている。
- `codd_gate_*.py` はパッケージ外の任意 sibling 部品とされている。

同じ方針の重複追記は行わなかった。また、対象文書は今回の書込許可範囲 `tools/agent-project/` の外にあるため編集していない。

## 検証

- §4 の「プラグイン境界（規範）」と §4.1 の「有効化」「永続化」「任意部品の可搬性」を完了条件と照合し、yaml/CLI だけで有効化することと、パッケージ内の直接結合を禁止することの両方を確認した。
- `git grep` で該当記述が `docs/designs/codd-gate-design.md` の 249、257〜258、301〜304、322〜323 行に存在することを確認した。
- `git diff --check` は成功した。
- `git status --short` は空で、作業ツリーに変更はない。

## 前提・未解決事項・範囲外

- 前提: 完了条件は新しい文章を追加することではなく、指定の方針を設計正典に明記して固定すること。既存記述が要求を具体的かつ重複なく満たしている場合は変更不要と判断した。
- 前提: 書込許可の「`tools/agent-project` 配下のみ」を優先し、範囲外の設計書は読み取りだけにした。
- 範囲外で見つけた問題: 現在の `agent_project/configfile.py` には `_apply_codd_gate_auto_wiring`、`doctor.py` と `model.py` には `codd_gate_*` の import が残っており、設計書に記載された受入コマンドは現時点では失敗する。元要求どおり、実装とテストは変更していない。

@followup 別の実装タスクで、§4.2 の受入条件に従ってパッケージ内の自動配線と `codd_gate_*` import を除去する。

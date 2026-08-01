# synth 統合結果

- t5〜t11 の累積差分を現物で照合し、概要を状態把握・有効化の唯一の導線、needs を失敗の意味と診断の表示に整理した。needs に重複していた設定ファイルボタンは削除した。
- t10 の契約境界を renderer と README へ反映し、自動探索した設定候補と agent-project の明示 `--config` による実効設定を同一視しない表示へ統一した。
- `codd-gate doctor` や単なる文字列言及をゲート失敗と誤分類しないよう、記録された canonical `codd-gate ... verify` を必要条件にした。
- YAML の引用符内 `#` と二重引用符の escaped newline を実効スカラーとして扱い、公式判定と逆転する境界を解消した。
- 既存コマンドの置換警告、未設定時のみの sibling CLI、JSON の既存 object 保持、`<root>` の置換案内を一貫させた。UI から設定・needs・done は変更しない。

## 検証

- `npm test`: 成功
- `node test/consistency-gate.test.js`: 15 件成功
- `node test/consistency-gate-ui.test.js`: 成功
- `node test/needs-gate-integration.test.js`: 8 件成功
- `node test/overview-ui.test.js`: 成功
- `node test/needs-diagnosis.test.js`: 12 件成功
- `git diff --check`: 成功
- 変更範囲: `tools/agent-dashboard/**` のみ

{"constraints":["agent-project が --config で解決した設定パスは現行 instance/status 契約から取得できないため、dashboard が表示できるのは自動探索した設定候補であり実効設定との一致は保証できない","regression_cmd と intake_cmd は汎用フックのため、設定有無と codd-gate 結線状態を別々に判定する","dashboard は一貫性ゲートの状態表示と人向け導線だけを担い、設定・needs・done を書き換えない"]}

@followup agent-project: instance/status 契約へ解決済み config path を追加後、dashboard の自動探索候補を実効設定表示へ置き換える。

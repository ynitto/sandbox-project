# synth 統合結果

## 結論

dashboard の一貫性ゲート可視化・有効化支援は HEAD `48a4353e6` に統合され、機械ゲート `gate2` は pass、現 HEAD での `npm test` と `git diff --check` も成功した。差分は `tools/agent-dashboard` 配下の 13 ファイル、1113 insertions / 10 deletionsで、agent-project 本体や UI からの状態書換は含まれない。

ただし最終受入れは条件付きである。UXレビューの parser 欠陥2件が残り、現在の `main` / `origin/main` は HEAD の祖先ではない。したがって依存成果の「リベース済み」という前提は採用せず、機能差分は成立しているが最新 main への rebase 完了および UX の全指摘解消は未達と判断する。

## 統合した成果

- データ経路: `agent-project.yaml` の `regression_cmd` / `intake_cmd` → `readToolConfig()` → `consistencyGateStatus()` → `readProject().consistencyGate` → `dashboard:project` → `state.project`。設定有無、codd-gate 結線状態、実効コマンドを読み取り専用モデルとして一度だけ判定する。
- UI: 概要に「結線済み／一部結線／未結線」と各フックの「設定あり／なし・結線済み／未結線・現在値」を表示する。未結線時は README と同じ YAML、`regression_cmd` 用 sibling CLI、`intake_cmd` の直接編集、設定ファイルを開く導線を示す。別コマンドは「設定あり・未結線」と区別する。
- needs: `needs/<id>.md` の `failure-*` → `parseNeeds()` → `renderNeedFacts()` の既存経路を保ち、失敗時の phase・要約・コマンドから codd-gate 由来と確認できる場合だけ、regression と task verify を区別して意味と intake 結線状態を補足する。
- テスト: main・UI・needs 統合の新規3テストを `npm test` に組み込み、t10 で overview の「別コマンド」「両方未結線」「既存 needs の件数・action・対応する導線」を追加固定した。対象テスト、全テスト、diff check は成功した。
- 文書: dashboard README に設定経路、失敗経路、有効化手順、読み取り専用境界を実装と同じ語彙で記載した。

## UXレビューとゲート証跡の統合

- gate1 / gate2: pass。最終 gate2 は 13ファイル・1113 insertions / 10 deletions、許可範囲内、対象テスト成功、clean、`{"ok":true,"issues":[]}`。
- UXレビュー: Request Changes。通常ケースの画面理解、有効化導線、needs 可読性、書込み境界は合格したが、`toolconfig.js` の malformed local JSON fallback と YAML folded scalar の段落消失により、例外ケースで結線済みと誤表示し得る。
- t8 / t9: 指摘は `project.js` / `renderer.js` の重複補正ではなく共有設定 reader の根本原因として扱い、両ファイルは変更しなかった。この判断は最小差分として妥当だが、UX指摘自体を解消したものではない。
- t10: production parser には触れず、既存実装の主要な表示回帰だけを20行のテストで固定した。よって gate2 pass と UXレビュー fail は矛盾せず、「指定機械条件は成功、既知の例外ケースは未解決」と整理する。
- rebase 証跡: t1 は rebase 未実施を明記しており、現時点でも `git merge-base --is-ancestor main HEAD` は不成立（`main...HEAD` は 360 behind / 17 ahead）。リベース済み成果としては認定できない。

{"constraints":["dashboard は一貫性ゲートの設定・失敗記録を読み取って案内するだけとし、設定・タスク状態・done を書き換えず、codd-gate も実行しない","regression_cmd / intake_cmd は汎用フックであるため、非空だけでは結線済みとせず、公式 codd-gate コマンド形との一致と設定有無を分離して表示する","結線表示は設定文字列上の状態であり、codd-gate の実在・互換性・実行成功を保証しない"]}

@followup `toolconfig.js` で最初に見つかった malformed local JSON を global 設定へ fallback せず解析失敗として扱い、結線済み判定と追記案を抑止する回帰テストを追加する。

@followup `parseFlatYaml()` の folded scalar で空行段落を PyYAML 相当に保持し、空行付き `>-` の回帰テストを追加する。

@followup 最新 `main` へ rebase し、既報の6競合候補を main の現構造優先で解消したうえで lint・全テスト・UXレビューを再実行する。

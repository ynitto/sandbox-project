# t16 最終受入照合

## 結論

**fail**。指定の完了条件コマンドは終了コード 0 で、codd-gate 系 115 tests は成功した。
また、`main...ap/sibling-163827` に `tools/agent-project/agent_project/` と dashboard の変更はない。
ただし README、GUIDE、`docs/designs/codd-gate-design.md` §4.1、実装の間に下記3件の不整合が残るため、
元要求「検出・yaml 注入・doctor 所見の置き場と README の有効化手順を一貫させる」は未達と判定した。

## 完了条件との1項目ずつの照合

1. sibling 自動検出レイヤを新境界へ追随: **pass**
   - `codd_gate_*.py` は `tools/agent-project/` 直下に残り、パッケージ配下へ移動していない。
   - `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
     は終了コード 0。
2. 利用手順を新境界へ追随: **fail**
   - README、GUIDE、設計、実装で永続化の書き手、生成時の適合性検査、doctor 所見の入口が一致していない。
3. agent_project パッケージ内への再結合なし: **pass**
   - `git diff --quiet main...HEAD -- tools/agent-project/agent_project` は終了コード 0。
   - `git diff --name-status main...HEAD` に同ディレクトリ配下の変更なし。
4. dashboard 変更なし: **pass**
   - `git diff --quiet main...HEAD -- tools/agent-dashboard tools/agent-project/dashboard` は終了コード 0。
   - `git diff --name-status main...HEAD` に dashboard を含むパスなし。
5. 指定完了条件: **pass（終了コード 0）**

```text
$ PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py' && grep -nE 'codd_gate_regression|regression_cmd|intake_cmd' tools/agent-project/README.md && ! grep -nE 'build_config.*メモリ上で自動|_apply_codd_gate_auto_wiring' tools/agent-project/README.md
...................................................................................................................
----------------------------------------------------------------------
Ran 115 tests in 0.034s

OK
276:  regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos <root>/repos.json'
277:  intake_cmd: 'codd-gate tasks --debt --repos <root>/repos.json'
281:  `--intake-cmd 'codd-gate tasks --debt …'` に渡す。`regression_cmd` は毎タスクの verify
282:  PASS 後、done 確定前に実行される。agent-project は `intake_cmd` が返す既存負債を修復タスクとして
286:  `regression_cmd` は生成ツールでも設定できる。既存の設定ファイルを `--config` で指定して、
290:  python3 tools/agent-project/codd_gate_regression.py \
294:  生成ツールは `regression_cmd` だけを冪等に更新する。`intake_cmd` は設定ファイルへ直接書く。
447:- **取り込みコマンド（intake_cmd）**: 外部の決定的ゲート/検出器を **watch の周期で pull** する汎用フック（push 型の
448:  inbox と対）。設定 `intake_cmd:`（CLI `--intake-cmd`）のコマンドをパス開始時と idle 中に `intake_interval`（既定
452:  持つ）。例: `intake_cmd: codd-gate tasks --debt`（doc/code/test 一貫性の負債を修復タスク化して自動返済）。
453:  > 外部 CLI を差し込める公式の口（verify/acceptance・regression_cmd・intake_cmd・inbox/enqueue・
exit=0
```

## 不足箇所・原因・必要な修正

### 1. yaml 永続化の書き手が一意ではない

- 設計 §4.1 314–327行は、yaml へ書く主体と経路を `codd_gate_regression.py` 一つだけと規定する。
- README 286–295行と GUIDE 127–135行も、生成ツールが `regression_cmd` だけを注入し、
  `intake_cmd` は人か install 手順が設定すると案内する。
- 一方、`codd_gate_wiring.py` 100–140行は `upsert_yaml_text()` / `apply_yaml_file()` を公開し、
  `regression_cmd` と `intake_cmd` の両方を永続化できる。設計のモジュール表 297行もこの第二の書き手を
  「yaml 冪等注入」として記載している。

原因は、境界回収で「一意の書き手」に戻した変更と、wiring 側へ yaml 注入 API を追加した変更が統合時に
併存したこと。

必要な修正は、`codd_gate_wiring.py` の yaml 書込 API を削除して
`codd_gate_regression.py` のみに寄せ、対応テストと設計表を直すこと。両 API を残す判断なら、逆に §4.1、
README、GUIDE の「一つだけ」「regression_cmd だけ」を改め、どちらを利用者が使うかを一意に説明する必要がある。

### 2. 生成 CLI の適合性検査に関する設計記述が実装と違う

- 設計 §4.1 317–322行は、生成 CLI が実在・バージョン・schema・capability を検査し、
  いずれか不適合なら書かないと記載する。300–307行も同趣旨。
- 実装 `codd_gate_regression.py` 233行は `detect_status()` を呼ぶだけである。
  `codd_gate_status.detect_status()` は実体の検出だけを行う簡易入口で、設計表 293行自身もその事実を記載している。
  生成 CLI は repos schema や capability を検査しない。
- README 295行の「見つからない場合は変更しない」は実装と合うが、設計のより強い保証とは合わない。

原因は、設計の旧保証を「現在地」の実装へ合わせ切れていないこと。

必要な修正は、現在の新境界を正とするなら §4.1 の生成 CLI に関する
「バージョン・schema・capability 不適合でも書かない」を「実体未検出なら書かない」へ修正すること。
強い保証を要求するなら、生成 CLI を `probe_wiring()` 相当の実測経路へ接続し、そのケースのテストを追加する。

### 3. doctor 所見の利用手順の正本が欠ける

- GUIDE 197–200行は direct CLI
  `python3 tools/agent-project/codd_gate_wiring.py --config ...` を案内し、README を有効化手順の正本とする。
- README の一貫性ゲート節 270–295行には direct wiring CLI も
  `hooks.wiring: codd_gate_wiring` も記載がない。
- 設計 §4.1 282–288行は direct CLI に加え、明示 hook の場合だけ package doctor に同じ所見が出ると記載する。
- 実装 `codd_gate_wiring.py` 283–321行の direct CLI は読み取り専用で GUIDE と一致するが、
  README からは結線診断の入口へ到達できない。

原因は、README の境界説明を短縮した際に doctor/結線診断の入口まで削除し、GUIDE の「README が正本」だけが
残ったこと。

必要な修正は、README の一貫性ゲート節へ direct wiring CLI を正準の診断手順として追記すること。
明示 hook を正式な第二経路として残すなら、その opt-in 設定例も README/GUIDE に揃えて記載する。
direct CLI のみに限定するなら、設計の package doctor 記述を削除する。

## 採用した前提・未解決事項・範囲外

- 指定作業ディレクトリの detached `HEAD` は `77389919` で、統合ブランチ先端
  `ap/sibling-163827` の `fdb66cb4` より古かった。「統合後の成果」はブランチ先端を指すと解釈し、
  規約どおり別の読み取り専用 worktree を provision して `fdb66cb4` を検証した。
- 評価は `main...ap/sibling-163827` の統合差分を対象にした。指定元 worktree、統合ブランチ、
  リポジトリ内ファイルは変更していない。
- `docs/designs/codd-gate-design.md` は許可フォルダ外だが、本タスクは読み取り照合だけを明示しているため参照した。
- 範囲外の agent_project パッケージ内再結合と dashboard 変更は見つからなかった。
- 修正は本タスクの役割外なので行っていない。

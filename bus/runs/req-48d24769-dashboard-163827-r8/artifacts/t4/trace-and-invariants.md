# t4: needs diagnosis / codd-gate / done 不変条件の調査

## 完了条件と採用した前提

- 完了条件は、(1) dashboard の設定・needs 表示経路、(2) codd-gate と回帰失敗要約の生成元、(3) agent-project の task/project done 判定を現物から追跡し、UI 変更後も守るべき可読性と状態境界を明文化すること、と解釈した。
- このタスクは参照のみであり、`tools/agent-dashboard` を含むリポジトリ内コードは変更しない。成果物だけを指定 artifact ディレクトリへ出す。
- 「結線済み」は設定文字列が既知の codd-gate コマンド形を満たすことだけを意味し、CLI の存在・schema 互換性・実行成功を意味しない。
- `needs/<id>.md` は agent-project の task status の投影であり、dashboard は表示モデルへ運ぶだけとする。done の正は agent-project 側にある。

## 追跡結果

### 1. 結線状態の表示経路

1. `agent-project.yaml` の `regression_cmd` / `intake_cmd` を dashboard の `readToolConfig()` が読む。探索は workspace の `.agents` 候補を優先し、最後にユーザーの agent home を見る。最優先 JSON が壊れている場合は global 値へ黙ってフォールバックしない。
2. `consistencyGateStatus()` が configured と wired を分離する。wired は agent-project の `codd_gate_wiring.py` と同じ語順条件で、regression は `codd-gate ... verify ... --base`、intake は `codd-gate ... tasks ... --debt`。`make smoke` 等の別コマンドは「設定あり・未結線」である。
3. `readProject().consistencyGate` → `dashboard:project` IPC → preload `readProject()` → renderer `reloadProject()` の `state.project` へ渡る。
4. 概要の `consistencyGateHtml()` は見出しを「結線済み / 一部結線 / 未結線」の3値で出し、各フックについて設定有無・結線有無・実コマンド・意味を別々に示す。未結線時だけ README と同じ設定行、既存設定ファイルを開く導線、regression 用 sibling CLI を案内する。UI は設定を書かず、ボタンも OS エディタで開くだけである。

根拠: `tools/agent-dashboard/src/features/agent-project/main/toolconfig.js:32-80`, `project.js:1597-1634,1641-1769`, `main/ipc.js:66-70`, `preload.js:7-10`, `src/renderer/renderer.js:818-833,1086-1197`, `sections/overview.js:349`。

### 2. codd-gate の意味

- `codd-gate verify --base REV` は差分を分類し、Amber（doc-stale / broken-ref / dangling-ref）を NG にする。Gray は `--strict`、Followup は `--strict-cross` のときだけ NG。exit 0=PASS、1=NG、2=使い方・前提不備である。
- agent-project への公式接続点は、task verify / charter acceptance、done 前の `regression_cmd`、周期取り込みの `intake_cmd`。intake は修復タスク供給であり done の根拠ではない。

根拠: `tools/codd-gate/codd-gate.py:1059-1128`, `tools/codd-gate/README.md:131-143,170-205`。

### 3. needs の失敗要約生成・表示経路

新しい task verify 失敗の正規経路:

1. agent-project の `diagnose_verify_failure()` が生の verify 出力を、summary / resolution / category / owner / command / workdir / exit_code / target に一度だけ正規化する。解釈不能は空のままにし、成功とは断定しない。
2. `_failure_record()` が phase / verdict と診断をまとめ、`_block(..., failure=...)` → `write_needs_file()` → `_failure_frontmatter()` により `failure-*` / `verify-verdict` を `needs/<id>.md` frontmatter へ1行ずつ書く。
3. dashboard `parseNeeds()` は `failure-summary` があれば `_failureFromFrontmatter()` でそのまま表示モデルへ移す。旧票だけ `_diagnoseFailure(why, detail)` で散文を後方互換解析する。
4. `renderNeedFacts()` は「検証失敗」要約、確認・対処、分類/対処対象/コマンド/作業場所/終了コード/確認対象の context を先に表示する。生の判断材料は同じカードの `<details><summary>判断材料を見る</summary>` に残す。
5. `needGateSource()` は、記録中に codd-gate がある場合だけ `regression` と task 自身の `verify` を分ける。現在の結線状態だけで過去の失敗原因を上書きしない。ゲート説明は既存の診断ブロックの後に追加される。

根拠: `tools/agent-project/agent_project/verify.py:573-653`, `mr.py:400-427`, `batch.py:94-112`, `needs.py:10-78,118-131`; dashboard `project.js:443-568,587-704`, `sections/needs.js:1295-1367,1419-1468`。

回帰ゲート失敗の現行経路には差異がある。`settle_task()` は task verify PASS 後に `regression_cmd` を `cfg.workdir` で実行し、NG なら直ちに `_block(...)` して `blocked` にするため done/review へ進まない。しかしこの呼び出しは `failure=` を渡していない。したがって現行 producer が作る回帰票は構造化 `failure-*` ではなく、`なぜ: 回帰検知...<regression_cmd>...<rmsg>` と task verify の evidence から dashboard の旧票 fallback が要約する。ゲート由来判定自体は `why` 内の `回帰検知` + codd-gate で成立するが、context の「コマンド」は task verify を拾う場合があり、実際に失敗した regression command と一致しない可能性がある。

根拠: `tools/agent-project/agent_project/mr.py:566-586` と dashboard `project.js:467-486`, `sections/needs.js:1307-1317`。

## UI 変更後も維持すべき可読性不変条件

1. 「検証失敗」ラベル、強調された1行要約、具体的な「確認・対処」、構造化 context、生の「判断材料を見る」を同じカード上に残す。ゲート案内で置換・隠蔽しない。
2. 実行未到達 (`verify-verdict=not_run`) を「検証失敗」と断定しない。解釈不能も空を成功扱いせず、生ログへの導線を残す。
3. regression と task verify を混同しない。前者だけ「完了前の `regression_cmd` が止めた」、後者は「タスク自身の検証」と説明する。
4. codd-gate 由来の断定は失敗時の記録（phase / command / why）に codd-gate がある場合だけ行う。現在 wired であることだけを過去失敗の根拠にしない。
5. `regression_cmd` / `intake_cmd` は設定有無と codd-gate 結線有無を分け、別コマンドの実値を隠さない。全体状態は3値を維持する。
6. wired は「設定文字列上の結線」であり、実行可否・互換性・成功を保証しないと明記する。intake も「正常に実行された場合に起票」と条件付きで示す。
7. 未結線導線は README と一致させる。存在しない設定ファイルに sibling CLI を勧めず、`intake_cmd` に注入 CLI が無いこと、JSON と YAML の形式差、cross-runtime path をコマンドへ直挿ししないことを守る。
8. 設定/コマンド文字列は外部入力として escape する。表示追加後も DOM の入れ子を壊さない。

## done 不変条件

1. dashboard は task status、project status、設定、done を直接書き換えない。一貫性ゲート UI の操作は表示と「設定ファイルを開く」だけに限定する。
2. task auto-done は `task.verify` が安定 PASSし、`regression_cmd` も PASSし、flake / no-progress / undiscriminating / protect / review 等のゲートに止められない場合だけ `_settle_done()` が確定する。
3. regression NG は `blocked` + needs を生成して done/review 分岐を通らない。UI からこれを緩和・承認済みに見せない。
4. review の done は、すでに verify PASS 済みの保持成果に対する人の approve 経路だけで確定する。MR 統合失敗時は review のままにする。
5. project は acceptance 全 PASSかつ改善ゼロでまず `converged` に止まり、人の milestone approve が `finalize_project()` を呼んで `accepted` と最終納品書を確定する。acceptance 未定義/合成不能や clone 失敗を PASS に倒さない。
6. `intake_cmd` は backlog 供給であり、task/project done を確定する経路に接続しない。

根拠: `tools/agent-project/agent_project/mr.py:318-381,520-661`, `commands.py:6-58,78-88`, `project.py:26-52,202-230,233-282,368-381,485-520`。

## 検証

- `npm test`: 成功（全 dashboard test。出力末尾まで fail 0）。
- `node test/needs-diagnosis.test.js`: 13/13 成功。
- `node test/needs-gate-integration.test.js`: 8/8 成功。
- `node test/consistency-gate.test.js`: 13/13 成功。
- `node test/consistency-gate-ui.test.js`: 成功。
- `git diff --check`: 成功。
- `git status --short`: 出力なし。指定 worktree は clean で、リポジトリ内の変更はない。

## 未解決・範囲外

- `@followup agent-project`: regression NG の `_block()` に構造化 failure record を渡し、`failure-phase=regression`、失敗した `failure-command=regression_cmd`、回帰の workdir/exit/summary/resolution を producer 側で確定する。現状は dashboard の散文 fallback に依存し、context command が task verify を指し得る。done 条件は変更せず、診断データだけを正規化する。
- `@followup integration-test`: 実際の agent-project 回帰 NG が生成した `needs/<id>.md` を dashboard `parseNeeds()` / `renderNeedFacts()` まで通す producer-consumer 統合テストを追加する。現行 dashboard テストの構造化 regression fixture は実 producer 経路を通っていない。
- t1 報告の main との競合/rebase は本タスクの参照範囲外。git 規約に従い checkout/rebase/commit/push は行っていない。

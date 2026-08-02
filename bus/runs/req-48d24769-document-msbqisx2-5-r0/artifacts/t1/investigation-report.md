# agent-project redaction 調査報告

## (a) 成果／サマリー

### 完了条件として採用した前提

この work は `[out_of_scope] コード変更、テスト追加` の調査タスクであるため、完了を次の 4 点と解釈した。

1. `tools/agent-project` 内だけを調べる。
2. shared state、`brief/`、`decisions/` の生成・共有経路を特定する。
3. 既存 redaction、関連テスト、CI コマンドを特定する。
4. 後続の実装・テスト追加が使う単一の共有検査契約と最小変更箇所を確定する。

リポジトリに `.codegraph/` は無かったため CodeGraph は利用せず、`rg` と対象ファイルの読解で追跡した。リポジトリ本体は変更していない。

### 結論

実装済みの redaction／sanitizer は存在しない。契約だけが `tools/agent-project/README.md:344-370` に先行定義され、置換記号は次の 4 個で確定済みである。

- `[REDACTED:TOKEN]`
- `[REDACTED:HOME]`
- `[REDACTED:PROMPT]`
- `[REDACTED:CREDENTIAL]`

変更対象となる単一の共有前検査境界は、**共有候補の文字列を分類する 1 個の検出器**である。全書込経路が同じ検出器を使い、入口に応じて次の動作だけを変えるのが最小構成になる。

- `brief/` と `decisions/` の追記直前: 検出結果を上記記号へ置換し、再検査してから書く。
- state-git の commit/push 直前: 同期除外後の共有候補を同じ検出器で再検査し、1 件でも残れば fail-closed で commit/push しない。

共有名前空間フラグメント方式のため、検出器は全フラグメントより先にロードされる `agent_project/_head.py` に置けば、新規抽象・依存・フラグメント登録なしで `brief.py`、`decisions.py`、`stategit.py` から再利用できる。

### 呼出経路

#### brief 生成・再注入

主要入口は `agent_project/brief.py:64 append_brief_item`。現在は `_norm_brief_item` 後の本文を `brief/<id>.md` へ直接追記し、同じ本文の先頭 80 文字を journal にも直接記録する（`brief.py:68-87`）。したがって置換は journal 記録より前に必要。

呼出元は以下。

- `commands.py:670 cmd_revise` の `feedback`
- `needs.py:541 ingest_feedback` の `feedback`
- `brief.py:124 capture_insight`（ノード発見、cohort、差し戻し系の共通捕捉）
- `capture_insight` は `brief.py:144 append_decision` にも同じ教訓を射影する

保存後は `brief.py:92 brief_context` → request 生成へ注入される。完了時は `brief.py:102 retire_brief` → `config.py:699 archive_task` により `archive/<id>.md` へ転記される。よって state-git だけの検査では commit 前に次 run と archive へ漏れる。

#### decisions 生成

単一の永続化入口は `agent_project/decisions.py:35 append_decision`。`context/action/reason/affects/learn/avoid` を組み立て、`decisions/<id>.md` へ直接追記する（`decisions.py:43-58`）。commands、needs、project、prioritize、flow、brief から呼ばれるため、各 caller ではなくこの関数で一度だけ置換する。

#### state repository の共有

通常同期は `loop.py:105 state_sync` → `state_git_for` → `DirectStateGit.sync`。`sync` は `stategit.py:950 _changed_targets` で同期除外後の変更対象を得て、`_initial_commit` または `_worktree_commit` で commit、その後 push する。通常経路の検査位置は **`_changed_targets()` の直後、commit 分岐の前**。

別経路として multi-node coordination が `coordination.py:79 state_transaction` → `DirectStateGit.transaction` を使い、一時 worktree を `mutate` 後に直接 `git add/commit/push` する（`stategit.py:177-218`）。通常 `sync` だけを守るとこの経路が迂回になるため、**transaction でも mutate 後・git add 前に同じ共有候補検出器を必ず呼ぶ**必要がある。

検査対象集合は `_changed_targets` と同じ規則（ドット始まりの segment、`flow-archive/`、`claims/` を除外）にする。エラーには元値・行本文を含めず、相対ファイルパスと区分だけを含める。

### 最小の後続変更案

- `agent_project/_head.py`: 4 記号、検出器、文字列置換、共有ツリー検査を追加（検出規則の単一正本）。
- `agent_project/brief.py`: `_norm_brief_item` 後、重複判定・書込・journal より前に置換。
- `agent_project/decisions.py`: block を作る入力群または完成 block を追記前に置換。
- `agent_project/stategit.py`: `sync` の targets 確定後と `transaction` の mutate 後に同じ検出器で fail-closed 検査。
- `tests/fixtures/privacy.json`: 実在しない token、POSIX/Windows home、明示的な raw-prompt sentinel、credential と無害な本文。
- `tests/test_privacy.py`: brief、decisions、通常 sync、transaction の契約テスト。

`capture_insight` や各 command caller 個別への guard は不要。入口関数を守る方が漏れと重複を減らす。

## (b) 検証内容と結果

- `rg` で `REDACTED|redact|sanitize|privacy` を実装・テスト配下に検索: redaction 実装・fixture・契約テストは 0 件。README の契約のみ。
- `append_brief_item`、`capture_insight`、`append_decision` の全 caller を検索し、上記の集約入口を確認。
- state-git の通常 `sync` と coordination の直接 `transaction` の 2 commit/push 経路を確認。
- 既存関連テスト:
  - `tests/test_delivery.py:37 TestRunBrief`（brief の追記、feedback/revise、request 注入、archive 転記）
  - `tests/test_decisions.py:66 TestDecisionRecords` 以降（decisions の生成・学習）
  - `tests/test_state_git.py:12 TestStateSyncBatching` 以降（commit/push、競合、transaction）
- ローカルで CI と同じ agent-project 全スイートを実行: `AGENT_FLOW_STUB_SLEEP_MAX=0 python -m unittest discover -s tools/agent-project/tests`、PASS（終了コード 0）。現時点では redaction テストが無いため、これは既存ベースラインの確認。
- GitHub Actions: `.github/workflows/ci.yml:31-32,56-61` が PR と main push で `tools/agent-project/tests` を `python -m unittest discover` する。後続で同ディレクトリへ `test_privacy.py` を追加すれば追加 CI 編集なしで redaction 失敗がジョブ失敗になる。
- `git diff -- tools/agent-project` と `git status --short -- tools/agent-project`: 変更なし。

## (c) 前提・未解決事項・範囲外の問題

### 採用した前提

- 「token」は認証 token の値を指す。`max_tokens` や分散 claim の `claim_token` というフィールド名だけを禁止すると既存機能が成立しないため、単語 `token` の単純一致にはしない。
- 任意の自然言語から「生プロンプト」を確実に推定することはできない。fixture では raw-prompt を識別可能な明示 sentinel／ラベル付き形式にし、その形式を契約化する。別の構造化入力フィールドを正典にする場合は後続実装時に契約を一つ選ぶ必要がある。
- home は実行ホストの `Path.home()` だけでなく、fixture の POSIX `/home/...`・`/Users/...` と Windows `C:\\Users\\...` を検査する。テスト自体に実ユーザーの home 値は書かない。
- state snapshot に直接混入した禁止値は自動修復せず同期を拒否する。これは README の fail-closed 契約に従う。

### 未解決事項

- README は `brief/decisions` の「置換不能時は追記しない・失敗を返す」とするが、`append_decision` は現在 DR id 文字列、`append_brief_item` は bool を返す。例外にするか戻り値契約を拡張するかは後続実装で決める必要がある。最小かつ fail-closed なのは専用例外で caller まで失敗を伝播する方式。
- `state_sync` は同期例外を journal に記録してループを継続する既存 best-effort 契約である。redaction 検査失敗でも commit/push は止まるが、CLI プロセス全体の終了コードは常に非ゼロになるとは限らない。契約テストは `DirectStateGit.sync/transaction` の失敗と remote HEAD 不変を直接検証するのが確実。
- 既存 Git 履歴、外部ツールの直接 push、CI 迂回は README 上も保証範囲外。

### 範囲外で見つけた点

`append_brief_item` は現在、本文の先頭 80 文字を journal に複製する。brief 本文だけを置換しても journal 生成より前でなければ漏れるため、必ず置換済み `body` を journal に渡す必要がある。journal 全体は state snapshot 再検査でも守るが、ローカルファイルへの早期漏出を避けるには入口側の順序が重要。

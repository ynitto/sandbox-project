# t8 成果報告: needs レンダリング回帰保護

## 成果／サマリー

コード変更は不要と判断した。現 HEAD の `tools/agent-dashboard` では、needs 詳細の単一レンダリング結合点 `renderNeedFacts()` が次の順序を維持している。

1. `need-diag` の「検証失敗」ラベルと `failureSummary` の強調表示
2. `failureResolution` の「確認・対処」
3. `failureContext` の分類・対処対象・コマンド・作業場所・終了コード・確認対象
4. codd-gate 由来の場合だけ追加する `need-gate` の状態・有効化案内
5. 既存の「理由」「概況」および詳細の判断材料

このため、新しい一貫性ゲート表示は既存の codd-gate／needs-diagnosis 失敗要約を置換せず、その後ろに追加される。通常の検証失敗、`consistencyGate` 未提供、失敗要約なしではゲート案内を出さず、既存表示を維持する。needs 内容、done 判定、公式契約フィールド、操作経路は変更していない。worktree のファイル変更はない。

## 検証内容と結果

- `node test/needs-gate-integration.test.js`: 8件成功。失敗要約・ラベル・context・詳細折り畳みを維持しながらゲート案内が共存すること、通常失敗を誤分類しないことを確認。
- `node test/needs-diagnosis.test.js`: 12件成功。codd-gate の失敗工程・失敗理由・生の `why`／`detail` を維持し、kind=`blocked`、decided=`false` のままであることを確認。
- `node test/needs-layout-ui.test.js`: 成功。needs 一覧・詳細レイアウトの既存契約を確認。
- `npm test`: 全テスト成功。
- 対象4ファイルの ESLint: 成功（`src/renderer/sections/needs.js`、上記3テスト）。
- `git diff --check -- tools/agent-dashboard`: 成功。
- `git status --short`: 変更なし。
- `npm run lint`: 既存の `src/features/cowork/main/cowork.js:590` の未使用変数 `cfg` 1件で失敗。本タスクと無関係なため変更していない。

## 採用した前提・未解決事項・範囲外

- 「単一レンダリング結合点」は、一覧にも使われる `needFailureViewModel()` の解析済み要約を受け、詳細の状況節を一括描画する `renderNeedFacts()` と解釈した。
- 「優先度」は、詳細で失敗要約・対処・失敗コンテキストをゲート案内より先に出す順序、および一覧で `failureSummary` を `why` より優先する既存規則を指すと解釈した。
- 「改行」は、生の判断材料を `mdToHtml()` で詳細折り畳みに保持し、失敗要約を別ブロックのまま表示する既存構造を指すと解釈した。
- 未解決は package 全体 lint の上記既存エラーのみ。
- @followup: `cowork.js:590` の未使用変数は別タスクで除去または利用意図を確認する。
- agent-project 本体、needs 内容更新、done 判定、設定書込み、公式契約外フィールド追加は範囲外として実施していない。

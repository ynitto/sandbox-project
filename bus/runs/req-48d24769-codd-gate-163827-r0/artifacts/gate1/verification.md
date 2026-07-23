# t1–t3 相互検証

verify=fail

## 判定

主要な責務境界の再導出は t1/t3 と一致した。ただし、正典と README は現行コードと不一致で、正典内の完了ゲートも宣言した境界全体を検査できない。t2 の「未解決事項なし」はこの不一致を見落としている。

## issues

1. `docs/designs/codd-gate-design.md:248-259,273-280,301-325` と `tools/agent-project/README.md:279-286` は、パッケージが `codd_gate_*` を import/自動配線せず、sibling 削除時も同一挙動とする。現行コードは `agent_project/configfile.py:201-229,376` で E2/E3 をメモリ上に自動注入し、`agent_project/doctor.py:287-320,528` が `codd_gate_wiring` を、`agent_project/model.py:494-515,527-557` が `codd_gate_debt` を import する。実装整理はスコープ外だが、後続タスクでこの3経路を汎用フックから除く必要がある。
2. `docs/designs/codd-gate-design.md:356-369` の必須完了ゲート `! git grep _apply_codd_gate -- tools/agent-project` は自動配線の2マッチしか見ない。`doctor.py` / `model.py` の `_codd_gate` と `import codd_gate_*` を残しても PASS するため、同文書 `:356-357` の「名指し・import・自動配線しない」を完了条件に固定できていない。`docs/designs/codd-gate-design.md:371-375` の厳密 grep を任意例ではなく必須受入にするか、同等に package 内の名指し/import を否定するゲートを追加すべき。
3. `docs/designs/codd-gate-design.md:289,309-314` と `tools/agent-project/README.md:286-287` は、生成ツールが version/schema/capability 不適合時に値を書かないとする。実際の `codd_gate_regression.py:180-182` は `detect_status()` のみを使い、`codd_gate_status.py:119-138` は実在すれば version/schema を既定で適合とし、capability も測定しない。設計を現状に合わせるか、後続実装で `detect_wiring()` 相当の完全な判定を生成 CLI に使わせる必要がある。
4. t2 `readme-consistency-gate-extract.md` の「未解決事項および範囲外で見つけた問題はない」は不正確。README `:279-286` は上記1の実装と直接矛盾する。t2 の結果に「README の抽出自体は正しいが、現行 package とは不一致」と根拠行を追記すべき。
5. (minor) t3 は「関連単体テスト4ファイル、72 tests」とするがファイル名を示さず、同成果物自身が関連 test として列挙する `test_codd_gate_{debt,detect,regression,routing,wiring}.py` は5ファイルある。5ファイル全部の再実行結果は81 tests PASS。実行対象を明記し、routing を除外したなら理由を書くべき。

## 独立検証結果

- `.codegraph/` は存在しないため、現物を `git grep` / `rg` と行番付き参照で再導出した。
- 必須 grep は `_apply_codd_gate` 2件で現時点 FAIL。厳密 grep も `configfile.py` / `doctor.py` / `model.py` に複数マッチし FAIL。
- codd-gate 関連単体テスト全5ファイルは81件すべて PASS。これらは現行挙動の回帰は保証するが、正典との一致は保証しない。
- worktree の未コミット差分は0。`main...HEAD` の差分は `tools/agent-project/README.md` 1ファイルのみで、許可範囲外の差分はない。

{"ok": false, "issues": ["design/README の非依存・非自動配線宣言と configfile/doctor/model が矛盾", "正典 §4.2 の必須 grep が package 内の名指し/import を見逃す", "生成 CLI の非互換 no-op 宣言と existence-only detect_status 実装が矛盾", "t2 が README と現行コードの不一致を未解決事項として落としている", "(minor) t3 のテスト対象と4ファイル/72件の対応が不明"]}

# t18 adversarial 境界整合性再検証

## 結論

**fail**。

指定完了条件は終了コード 0、codd-gate 系 111 tests は成功した。通常入力では YAML 書込実装が
`codd_gate_regression.py` 一つだけであること、wiring CLI が読み取り専用であること、生成 CLI の
冪等性、`agent_project/` と dashboard に差分がないことも確認した。

ただし adversarial 入力で生成 CLI が既存 YAML を破壊するケースと、存在しない明示実体を検出成功と
扱って設定を書き込むケースを再現した。また、direct CLI を唯一の正準入口とする文書に対し、
明示 hook から package doctor へ入る経路を実装・テストが維持している。agent-reviewer の4観点中
3観点が `REQUEST_CHANGES` のため、集約規則どおり fail とする。

## 検証内容と結果

### 指定完了条件

次をそのまま実行し、終了コード 0。

```text
PYTHONPATH=tools/agent-project python3 -m unittest discover -s tools/agent-project/tests -p 'test_codd_gate_*.py' &&
grep -nE 'codd_gate_regression|regression_cmd|intake_cmd' tools/agent-project/README.md &&
! grep -nE 'build_config.*メモリ上で自動|_apply_codd_gate_auto_wiring' tools/agent-project/README.md

Ran 111 tests in 0.037s
OK
```

### 境界・差分

- `git diff --quiet main...HEAD -- tools/agent-project/agent_project tools/agent-dashboard tools/agent-project/dashboard`
  は終了コード 0。パッケージ内再結合と dashboard 変更なし。
- `! git grep -nE '_apply_codd_gate|_codd_gate|import codd_gate' -- tools/agent-project/agent_project`
  は終了コード 0。パッケージに codd-gate 固有 import・自動配線なし。
- `codd_gate_*.py` の `write_text()` は `codd_gate_regression.py` だけ。
  `codd_gate_wiring.py` に `upsert_yaml_text` / `apply_yaml_file` は存在しない。
- t17 commit の変更は、t17 が明示した文書3件、実装2件、対応テスト2件の計7件。
- `git diff --check HEAD^..HEAD` は終了コード 0。作業ツリーは clean。
- 一時ディレクトリ上の実行で、wiring CLI が設定を変更しないこと、生成 CLI の初回
  `changed=true`・再実行 `changed=false`、既存 `intake_cmd` の保持を確認。

### agent-reviewer 集約

| perspective | 判定 | Critical | Warning | Suggestion |
|---|---:|---:|---:|---:|
| functional | REQUEST_CHANGES | 0 | 4 | 0 |
| architecture | REQUEST_CHANGES | 0 | 2 | 0 |
| test | REQUEST_CHANGES | 0 | 2 | 0 |
| document | LGTM | 0 | 0 | 0 |

文書観点では README・GUIDE・設計 §4.1 の正準説明自体は一致している。阻害事項は実装・テスト・
書込許可との不一致である。

## 阻害事項と修正方法

### 1. ネストした同名 YAML キーを破壊する

- 場所: `tools/agent-project/codd_gate_regression.py:107-130`
- 原因: `_key_pattern()` が `^[ \t]*regression_cmd:` を許すため、ネストしたキーも
  トップレベルキーとして一致する。置換行はインデントを持たない。
- 再現結果:

```yaml
root: .agent-project
nested:
  regression_cmd: keep-nested
agent_cli: codex
```

が次へ変わった。

```yaml
root: .agent-project
nested:
regression_cmd: 'codd-gate verify --base "$KIRO_BASE_REV" --repos .agent-project/repos.json'
agent_cli: codex
```

- 修正方法: `regression_cmd` と anchor は行頭から始まるトップレベルキーだけに一致させる。
  ネストした同名キーを保持する回帰テストを `test_codd_gate_regression.py` に追加する。

### 2. 存在しない `--codd-gate` を実体検出成功として扱う

- 場所: `tools/agent-project/codd_gate_detect.py:25-35`,
  `tools/agent-project/codd_gate_regression.py:209-225`
- 原因: `resolve_codd_gate(explicit)` が明示文字列の実在を確認せず argv として返し、
  `detect_status()` はそれだけで `usable=True` にする。
- 再現結果: `--codd-gate /definitely/missing/codd-gate` で終了コード 0、
  `usable=true`、`changed=true` となり `regression_cmd` が書き込まれた。
  README 295行と設計 §4.1 の「実体未検出なら書き込まない」に反する。
- 修正方法: 共通入口 `resolve_codd_gate()` で明示パスの実在・実行可能性を検証し、
  未検出なら `None` を返す。生成 CLI が `EXIT_UNUSABLE` で YAML を保持するテストを追加する。

### 3. package doctor への第二経路を実装・テストが維持する

- 場所: `tools/agent-project/codd_gate_wiring.py:16-25,219-228`,
  `tools/agent-project/tests/test_codd_gate_wiring.py:286-381`
- 内容: README 298-306行、GUIDE 197-200行、設計 §4.1 282-298行は direct read-only CLI を
  唯一の正準入口とする。一方、`detect_wiring = probe_wiring` と
  `doctor_findings = render_findings` を公開し、`hooks.wiring: codd_gate_wiring` から package doctor
  へ到達できる。テストもこの経路を積極的に保証する。
- 修正方法: direct CLI のみにするなら両 alias、package doctor 経路のソース説明、
  その結線を保証するテストを削除し、代わりに wiring CLI の JSON と設定ファイル不変を一気通貫で
  テストする。互換経路を残すなら、3文書の「唯一」を改め第二経路を明記する。

### 4. 書込許可範囲外の変更

- 場所: `docs/designs/codd-gate-design.md`
- 内容: t17 は同設計書の更新を明示する一方、workspace 契約は
  「変更してよいのは `tools/agent-project` 配下のみ」と限定する。t17 は前者を例外と解釈したが、
  許可拡張は明示されていない。
- 修正方法: 設計書変更への明示的な許可を得るか、許可された別タスクへ分離する。

### 5. 永続化境界のテストが別名書込を検出できない

- 場所: `tools/agent-project/tests/test_codd_gate_wiring.py:107-110`
- 内容: 削除した2属性の不在だけを検査するため、別名の書込 API や `main()` 直書きは検出しない。
- 修正方法: wiring CLI 実行前後の内容と mtime の不変を検証し、codd-gate モジュール群の書込箇所が
  regression 側だけであることを固定する。

## 前提・未解決事項・範囲外

- 「指定完了条件」は t16 に記載された連結コマンドを指すと解釈した。
- 評価対象は t17 commit `5732cb204010b56babb521e3c80cd855817373be` とし、禁止範囲の確認だけ
  `main...HEAD` へ広げた。
- 「正準入口が一意」は推奨手順の文言だけでなく、実装とテストが正式に保証する診断入口も一致すること、
  と adversarial に解釈した。
- `docs/designs/codd-gate-design.md` の許可競合は t17 報告ですでに発見済みのため、新たな恒常的制約
  には数えない。
- リポジトリは変更していない。成果物はこの報告だけ。

<!-- verdict-json -->
```json
{
  "skill": "agent-reviewer",
  "verdict": "REQUEST_CHANGES",
  "blocking": true,
  "perspectives_executed": ["functional", "architecture", "test", "document"],
  "perspective_results": [
    {
      "perspective": "functional",
      "verdict": "REQUEST_CHANGES",
      "blocking": true,
      "severity_summary": {"critical": 0, "warning": 4, "suggestion": 0}
    },
    {
      "perspective": "architecture",
      "verdict": "REQUEST_CHANGES",
      "blocking": true,
      "severity_summary": {"critical": 0, "warning": 2, "suggestion": 0}
    },
    {
      "perspective": "test",
      "verdict": "REQUEST_CHANGES",
      "blocking": true,
      "severity_summary": {"critical": 0, "warning": 2, "suggestion": 0}
    },
    {
      "perspective": "document",
      "verdict": "LGTM",
      "blocking": false,
      "severity_summary": {"critical": 0, "warning": 0, "suggestion": 0}
    }
  ],
  "aggregated_blocking_issues": [
    {
      "from_perspective": "functional",
      "severity": "Warning",
      "summary": "ネストした同名キーをトップレベル設定として破壊的に置換する",
      "location": "tools/agent-project/codd_gate_regression.py:107"
    },
    {
      "from_perspective": "functional",
      "severity": "Warning",
      "summary": "存在しない明示 codd-gate パスでも YAML を更新して成功終了する",
      "location": "tools/agent-project/codd_gate_detect.py:25"
    },
    {
      "from_perspective": "architecture",
      "severity": "Warning",
      "summary": "direct CLI 唯一という文書に反して package doctor 経路が残る",
      "location": "tools/agent-project/codd_gate_wiring.py:219"
    },
    {
      "from_perspective": "architecture",
      "severity": "Warning",
      "summary": "許可された tools/agent-project 外の設計書を変更している",
      "location": "docs/designs/codd-gate-design.md:282"
    },
    {
      "from_perspective": "test",
      "severity": "Warning",
      "summary": "wiring CLI の read-only 性と書込主体一意性をテストが固定していない",
      "location": "tools/agent-project/tests/test_codd_gate_wiring.py:107"
    }
  ]
}
```
<!-- verdict-json -->

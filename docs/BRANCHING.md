# BRANCHING — {{ORG_NAME}} ブランチ・コミット・PR 規約

組織共通のルール。各リポジトリの `CONTRIBUTING.md` から本書を参照する。

> **原則**: 組織（{{ORG_NAME}} Organization）の設定を上位とし、本規約はそれを補足・実装するものとして位置づける。ラベル・デフォルトブランチ名等は Organization Repository defaults を仕様とする。

## 1. ブランチ戦略

### main

- Organization デフォルトブランチとして `main` を使用
- 常にデプロイ可能状態（CI green）
- **直接 push 不可**（Organization Ruleset で強制）
- 全変更は PR 経由
- **Squash merge 一本**、linear history 維持

### 作業ブランチ命名

`<type>/<short-description>` 形式：

| プレフィックス | 用途 | 例 |
|---|---|---|
| `feat/` | 新機能追加 | `feat/memory-tag-search` |
| `fix/` | バグ修正 | `fix/auth-error-message` |
| `refactor/` | リファクタリング（機能変更なし） | `refactor/extract-kv-adapter` |
| `docs/` | ドキュメントのみ | `docs/update-architecture` |
| `test/` | テストのみ | `test/store-coverage` |
| `chore/` | 依存更新・設定 | `chore/bump-wrangler` |
| `ci/` | CI/CD 設定 | `ci/add-codeql` |

### 使わないブランチ

- `main` / `master` / `develop` は **削除不可**（Organization Ruleset / GitHub MCP API で強制）
- `claude/intelligent-euler-*` 形式のセッション固有ブランチは使わない（上記規約に従う）
- 作業ブランチはマージ後即削除

## 2. コミットメッセージ

### 形式

[Conventional Commits](https://www.conventionalcommits.org/) 準拠：

```
<type>(<scope>): <subject>

<body 任意>
```

| type | 用途 |
|---|---|
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメント |
| `refactor` | リファクタリング（動作不変） |
| `test` | テスト |
| `ci` | CI/CD |
| `chore` | 雑務（依存更新、設定等） |
| `perf` | パフォーマンス |
| `build` | ビルド・パッケージング |

### scope（任意）

機能領域を中括弧で入れる：`feat(auth): ...`、`fix(store): ...`。小規模な変更では省略可。

### subject ルール

- 50 文字以内推奨
- 命令形（「add」、「追加」。「added」、「追加した」は不可）
- ピリオドで終わらない
- 日本語可（プロジェクト方針として日本語 OK）

### Claude が生成したコミット

末尾もしくはコミット subject に「（Claude による）」または「Claude による初版」等を含める。

**含めない**：
- `Co-Authored-By:` タグ
- `https://claude.ai/code/...` URL タグ
- その他 LLM ツール名 / セッション ID

### 例

```
feat(memory): tag による AND 検索を追加
fix(auth): Bearer 検証で長さチェック追加（タイミング攻撃対策）
docs: REQUIREMENTS に IP ban 防止ポリシー追加（Claude による改訂）
chore(deps): wrangler を v4 へアップデート
ci: verify-secrets ワークフローに account_id 検証追加
```

## 3. Pull Request

### PR タイトル

コミットメッセージと同じ規約。複数コミットを squash する際の最終形を意識して付ける。

### PR 本文

`.github/PULL_REQUEST_TEMPLATE.md` で雛型提供（テンプレート利用者が自リポジトリで作成）。最低限：

```markdown
## Summary
- 何を変えたか（1-3 行）

## Why
- なぜ変える必要があったか

## Test Plan
- [ ] npm run typecheck
- [ ] npm test
- [ ] (手動テスト項目)

## 関連
- Closes #XX
```

### ラベル（Organization デフォルトの 15 個）

[組織設定](https://github.com/organizations/{{ORG_NAME}}/settings/repository-defaults) で以下 15 個を全リポジトリに自動付与する想定。PR には適切なものを選んで必須付与：

| ラベル | 意味 |
|---|---|
| `breaking-change` | 後方互換性を壊す変更 |
| `bug` | Something isn't working |
| `dependencies` | 依存パッケージ更新 |
| `documentation` | Improvements or additions to documentation |
| `duplicate` | This issue or pull request already exists |
| `enhancement` | New feature or request |
| `good first issue` | Good for newcomers |
| `help wanted` | Extra attention is needed |
| `infra / ci` | デプロイ・CI 設定変更 |
| `invalid` | This doesn't seem right |
| `question` | Further information is requested |
| `refactor` | 機能変更を伴わないコード整理 |
| `requirements` | 要件定義の議論・更新 |
| `security` | セキュリティ関連の修正 |
| `wontfix` | This will not be worked on |

#### 変更タイプ → ラベル マッピング

| 変更 | 主ラベル (必須) | 付隱 (任意) |
|---|---|---|
| 新機能 | `enhancement` | `requirements` |
| バグ修正 | `bug` | |
| 要件変更 | `requirements` | `documentation` |
| ドキュメント | `documentation` | |
| リファクタリング | `refactor` | |
| セキュリティ修正 | `security` | `bug` (修正型) |
| 依存更新 | `dependencies` | `security` (脆弱性修正時) |
| CI / インフラ | `infra / ci` | |
| 後方互換性破壊 | `breaking-change` | + 上記いずれか |

### マージ方式

- **デフォルト：Squash merge**（linear history 維持）
- merge commit / rebase merge は使わない
- マージ後は作業ブランチを削除

### レビュー

- **個人運用フェーズ**: セルフレビューで OK（Reviewer フェーズで Claude にレビュー依頼可）
- 複数人運用になったら approving review 1+ 必須化

## 4. Branch Protection / Organization Ruleset

### 推奨：Organization Ruleset（複数リポジトリ一括制御）

個別リポジトリの Branch Protection ではなく、**Organization Ruleset で一括設定**を推奨。すべてのリポジトリに同一規則を適用できる、テストモード (Evaluate) で安全に試せる、一元管理・一覧可能。

**手順**：

1. `https://github.com/organizations/{{ORG_NAME}}/settings/rules` → **New ruleset → New branch ruleset**
2. ルールセット名：`main-protection-all-repos`
3. **Enforcement status**: 最初は `Evaluate`、確認後 `Active` に変更
4. **Target repositories**: All repositories または Selected（対象リポジトリを選択）
5. **Target branches**: Include default branch (`main`)
6. **Rules** で以下にチェック：

| ルール | 詳細 |
|---|---|
| Restrict deletions | ☑ |
| Require linear history | ☑ |
| Require a pull request before merging | ☑ (Required approvals: 0、Dismiss stale = ☑) |
| Require status checks to pass | ☑ (status check 名: 各プロジェクトの CI ジョブ名を初回 CI 実行後に追加) |
| Block force pushes | ☑ |
| Require code scanning results | (任意) |

7. **Bypass list**: 個人運用フェーズでは空 (もしくは Organization admin のみ)

### フォールバック：個別リポジトリ Branch Protection

Organization Ruleset が使えないケースやリポジトリ個別の例外には Settings → Branches → Add rule で各リポジトリ設定。値は上記 6 ルールと同じ。

## 5. タグ運用（Semver）

### 形式

`vX.Y.Z`（`v` プレフィックス必須）

| 種別 | 内容 |
|---|---|
| MAJOR (X) | 後方互換性なしの破壊変更 |
| MINOR (Y) | 後方互換性ある機能追加 |
| PATCH (Z) | バグ修正、内部改善 |

### 例

- `v0.1.0` — 初版
- `v0.1.1` — CI 修正等のパッチ
- `v0.2.0` — OAuth 対応等の機能追加
- `v1.0.0` — 安定版リリース

### タグでのデプロイトリガー（例）

プロジェクトのデプロイ先に応じて、`v*` タグ push をトリガーに `.github/workflows/deploy.yml` 等を起動する構成を採用する。例：

- **Cloudflare Workers 系**：`v*` push → `deploy.yml` → Cloudflare へデプロイ
- **コンテナ系**：`v*` push → `image.yml` → コンテナレジストリへ push
- **ドキュメント / 設定のみのリポジトリ**：タグ作成は任意、デプロイなし

> テンプレート利用者は自プロジェクトのデプロイ方針に合わせてこの節を書き換えること。

### トリガー手順

```bash
git tag v0.1.1
git push origin v0.1.1
```

または GitHub UI から手動 release 作成。

## 6. Claude による自動コミットの扱い

Claude Code 経由で main へ直接 push する場合（雛形作成・要件定義など初期立ち上げフェーズに限定）：

1. **初期セットアップ限定**：Organization Ruleset を一時的に `Evaluate` または Disabled に
2. コミットメッセージに「（Claude による）」を含める
3. 雛形が固まったら Ruleset を `Active` に、以降は PR 必須

雛形期間を越えたら、Claude も「作業ブランチを切る → 作業 → PR 作成」フローに従う。GitHub MCP server の `create_branch` / `push_files` / `create_pull_request` ツールを使う。

## 7. 開発コンテクストルール（overview + 1 リポジトリ）

> 複数リポジトリを横断する組織で、別途「overview（メタ）リポジトリ」を運用する場合のみ適用。単一リポジトリ運用なら本節は読み飛ばしてよい。

個別プロジェクトを開発する際は、**Claude は overview リポジトリと対象 1 リポジトリの 2 つをコンテクストに入れて動く**。以下のルールで重複・コンテクスト腰折れを避ける。

### 7.1 読む順番（セッション開始時）

Claude は以下の順でドキュメントを読む：

1. `<overview>/docs/REQUIREMENTS.md` — システム全体
2. `<overview>/docs/BRANCHING.md` — 議事規約 (本書)
3. `<overview>/docs/ROLES.md` — 7 ロール定義
4. `<overview>/docs/CODING_STANDARDS.md` — 実装規約
5. `<overview>/docs/SECURITY_BASELINE.md` — セキュリティ基準
6. `<対象>/docs/REQUIREMENTS.md` — 単体要件
7. `<対象>/docs/ARCHITECTURE.md` — (あれば)
8. `<対象>/docs/DECISIONS.md` — (あれば)

他リポジトリのディレクトリは見ない（コンテクストの分離）。

### 7.2 コンテンツの置き場所ルール

| 内容 | 置き場所 |
|---|---|
| Organization 設定 (ラベル / Ruleset / デフォルトブランチ名) | **Organization Settings (GitHub UI)** |
| 全リポジトリ共通の規約 / 設計原則 | **overview リポジトリ** |
| ツール名規約 / コミット規約 | **overview リポジトリ** |
| TypeScript / コーディング規約 | **overview リポジトリ** |
| セキュリティ基準 | **overview リポジトリ** |
| 該当プロジェクトのツール仕様 | **対象リポジトリ** |
| 該当プロジェクトのコード | **対象リポジトリ** |
| ADR (該当プロジェクトの設計選択) | **対象リポジトリ** (`docs/DECISIONS.md`) |
| システム全体に関わる ADR | **overview リポジトリ** (`docs/DECISIONS.md`) |

### 7.3 重複避止

- 他リポジトリのツール仕様を overview にそのまま記載しない（リンクだけ）
- 個別リポジトリの REQUIREMENTS に BRANCHING / CODING_STANDARDS の内容をコピペしない（リンクだけ）
- 1 つの事柄は 1 つの場所にだけ記述
- Organization で設定されているもの (ラベル一覧 / デフォルトブランチ) は docs に詳細記述せず、GitHub UI へリンク

### 7.4 クロスカッティング変更のフロー

複数リポジトリに影響する変更をしたいとき：

1. まず **overview で規約・規格を更新** (本書 / ROLES / CODING_STANDARDS 等)
2. 各リポジトリで順次適応 PR (1 リポジトリスコープで作業)
3. overview の PR を先にマージしてから各リポジトリの PR をマージ

### 7.5 作業範囲ルール (Single Concern)

- 1 PR = 1 変更起点 (feat / fix / refactor を混ぜない)
- 実装とドキュメント更新は同じ PR に含めてよい (分けるほうが逆に意図不明になるところだけ)
- 依存追加した「ついでに修正」は別 PR に切り出す (レビューで観点を見やすくする)

### 7.6 セッション開始チェックリスト

Claude は作業開始時に以下を確認：

- [ ] 現在のロールは? (Analyst / Architect / ...)
- [ ] 進めるべきフェーズの「開始条件」を満たしているか? (ROLES.md 参照)
- [ ] 対象リポジトリの該当冲頭ドキュメントを読んだか?
- [ ] CODING_STANDARDS / SECURITY_BASELINE に意識があるか?
- [ ] 作業ブランチを切るか、初期セットアップ期間として main 直 push か?

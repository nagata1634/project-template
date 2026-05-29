# CLAUDE.md — {{ORG_NAME}} / {{REPO_NAME}} セッション規約

> Claude (claude.ai / Claude Code) で本リポジトリに対し作業する際の鉄則。
> 組織共通の規約があればそれを上位とし、本書はリポジトリ固有の追記として併読する。

## 上位ドキュメント（参照順）

1. [BRANCHING.md](docs/BRANCHING.md) — ブランチ・コミット・PR 規約
2. [ROLES.md](docs/ROLES.md) — 7 ロール開発プロセスと完了条件
3. [CODING_STANDARDS.md](docs/CODING_STANDARDS.md) — TS / コミット / テスト規約
4. [SECURITY_BASELINE.md](docs/SECURITY_BASELINE.md) — S1〜S12 セキュリティ基準
5. [MAINTENANCE.md](docs/MAINTENANCE.md) — ブランチ運用自動化・滞留クリーンアップ
6. [ORG_RULESET.md](docs/ORG_RULESET.md) — Organization Ruleset 設定手順

## ブランチ運用の鉄則（厳守）

1. **PR マージ直後に作業ブランチを削除する**
   - GitHub MCP の `merge_pull_request` 等でマージ後、`delete_branch` 系ツールがあれば即座に呼ぶ
   - ツールが無い場合は、削除コマンドまたは GitHub UI 手順を即座にユーザーに提示し、忘れない仕組みを最初の commit で導入する
2. **新規ブランチ作成前に既存ブランチを `list_branches` で確認する**
   - `main` / `master` / `develop` 以外のブランチが滞留していたらクリーンアップ提案を先にする
3. **`main` / `master` / `develop` には絶対に直 push しない**
   - branch protection 未設定でもルール上の主ブランチには直 push しない
4. **`claude/*` 形式のセッション固有ブランチは作らない**
   - ブランチ名は必ず `<type>/<short-description>` 形式（BRANCHING.md §1）

## ブランチ命名規則（BRANCHING.md §1 と同じ）

| プレフィックス | 用途 |
|---|---|
| `feat/`     | 新機能追加 |
| `fix/`      | バグ修正 |
| `refactor/` | リファクタリング（動作不変） |
| `docs/`     | ドキュメントのみ |
| `test/`     | テストのみ |
| `chore/`    | 依存更新・雑務 |
| `ci/`       | CI/CD 設定 |

## 新規プロジェクト初期セットアップの必須作業

新規リポジトリで作業を始める場合、**最初の PR** に以下を含める：

1. `.github/workflows/auto-delete-merged.yml`（マージ時自動削除）
2. `.github/workflows/cleanup-stale-branches.yml`（手動一掃）
3. `.github/workflows/setup-branch-protection.yml`（main 保護）
4. `docs/MAINTENANCE.md`（運用手順）

雛形は本テンプレートの `.github/workflows/` および `docs/MAINTENANCE.md` を参照。

## 既存プロジェクト介入時の最初の作業

`list_branches` で滞留を確認 → 多ければ：

1. ユーザーに「マージ済みブランチが N 個滞留しているので最初にクリーンアップ workflow を入れてよいか」確認
2. 同意があれば 3 ワークフローを最初の PR として投入
3. ユーザーに `workflow_dispatch` で「Cleanup stale branches」と「Setup branch protection」を順に実行してもらう

## セルフチェック（PR を作る前 / マージ後 必ず実施）

- [ ] 作業ブランチ名は規約に合っているか
- [ ] マージ後、ブランチが消えているか確認した
- [ ] 自動削除ワークフローが入っているか確認した（無ければ追加）
- [ ] `claude/*` 形式のブランチを残していないか

## コミットメッセージ規約（BRANCHING.md §2 と同じ）

- Conventional Commits 形式 `<type>(<scope>): <subject>`
- 末尾に「（Claude による）」を含める
- **`Co-Authored-By:` タグ・`https://claude.ai/code/...` URL タグは含めない**

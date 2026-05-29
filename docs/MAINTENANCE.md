# MAINTENANCE — ブランチ運用自動化

> 全リポジトリ共通のブランチ滞留対策・main 保護の自動化方針。
> 各リポジトリの `.github/workflows/` にコピーして使用する。

## 提供ワークフロー（3 種）

### 1. `auto-delete-merged.yml` — マージ時自動削除

- PR が **squash merge / merge** された瞬間に作業ブランチを自動削除
- 保護対象 (`main` / `master` / `develop`) と `dependabot/*` は対象外
- 同一リポジトリ内 PR のみ削除（フォーク PR は無触）

### 2. `cleanup-stale-branches.yml` — 一掃用

- `workflow_dispatch` で手動実行（GitHub UI > Actions > Run workflow）
- `dry_run: true` (default) で削除対象一覧のみ出力
- `dry_run: false` で **main にマージ済み (ahead_by == 0)** のブランチのみ実削除
- 未マージブランチは触らない（誤削除防止）

### 3. `setup-branch-protection.yml` — main 保護自動適用

- `workflow_dispatch` で手動実行
- main に対し以下のルールを適用:
  - PR 必須（approving review 0、ただし dismiss_stale_reviews ON）
  - linear history 必須（squash merge と整合）
  - force_push / deletions 禁止
  - status checks（リポ固有のチェック名は別途設定）

## 適用手順

新規リポジトリ:

```bash
# 3 ワークフローをコピー
mkdir -p .github/workflows
cp /path/to/project-template/.github/workflows/auto-delete-merged.yml .github/workflows/
cp /path/to/project-template/.github/workflows/cleanup-stale-branches.yml .github/workflows/
cp /path/to/project-template/.github/workflows/setup-branch-protection.yml .github/workflows/

# 初回 PR で投入、マージ後に手動で実行
# 1. Actions > "Setup branch protection (main)" > Run workflow
# 2. Actions > "One-time cleanup of stale branches" > Run workflow (dry_run=false)
```

既存リポジトリ:

1. `list_branches` で滞留を確認
2. 3 ワークフローを `chore/branch-maintenance-workflows` 等のブランチで導入する PR
3. マージ後 `Setup branch protection` を 1 回実行
4. `Cleanup stale branches` を dry_run=true で確認 → dry_run=false で実削除

## Organization Ruleset との関係

- `setup-branch-protection.yml` は **個別リポジトリの** branch protection 設定
- Organization Ruleset（[ORG_RULESET.md](ORG_RULESET.md)）が有効な場合、Ruleset が優先される（cumulative）
- 推奨は Organization Ruleset で一括管理 + 本 workflow は個別フォールバックとして温存

## 失敗時のフォールバック

- `auto-delete-merged.yml` が失敗してもマージ自体には影響しない（自動削除のみ skip）
- 削除に失敗した PR はログに残るので、後で `cleanup-stale-branches.yml` で回収
- API 権限不足の場合 `permissions: contents: write` が yaml に書かれているか確認

## 既知の制約

- フォーク PR のブランチは削除しない（リポジトリ間ではなく fork 側のため API 権限なし）
- GitHub Apps から merge した PR の場合、`actions/github-script` のトークンが head ブランチへの write 権限を持たない場合がある → その時は `secrets.GHA_PAT` を使う運用に切り替え
- `setup-branch-protection.yml` には `administration: write` が必要。Organization Ruleset を使っている場合は不要

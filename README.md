# project-template

汎用的な開発設定（ブランチ運用・コーディング規約・セキュリティ基準・7 ロール開発プロセス）の雛形。
新規リポジトリ立ち上げ時の初期セットアップを最小化することを目的とする。

## 含まれるファイル

### GitHub ワークフロー / 設定
- `.github/workflows/auto-delete-merged.yml` — PR マージ時に作業ブランチを自動削除
- `.github/workflows/cleanup-stale-branches.yml` — 滞留ブランチの一掃（手動実行）
- `.github/workflows/setup-branch-protection.yml` — main 保護を自動適用
- `.github/workflows/markdownlint.yml` — Markdown lint（PR / push 時）
- `.github/dependabot.yml` — GitHub Actions 依存の monthly 更新
- `.markdownlint-cli2.yaml` — Markdown lint 設定（日本語ドキュメント向け）

### ドキュメント
- `CLAUDE.md` — Claude Code セッション規約（鉄則・ブランチ命名・セルフチェック）
- `docs/BRANCHING.md` — ブランチ・コミット・PR 規約
- `docs/CODING_STANDARDS.md` — TypeScript / コミット / テスト規約
- `docs/MAINTENANCE.md` — ブランチ運用自動化の解説と適用手順
- `docs/ORG_RULESET.md` — Organization Ruleset 設定手順
- `docs/ROLES.md` — 7 ロール開発プロセス（Analyst → ... → SecurityAuditor）
- `docs/SECURITY_BASELINE.md` — S1〜S12 セキュリティ基準

## 使い方

### a) ファイル単位でコピー

必要なファイルだけを新規リポジトリにコピー：

```bash
git clone https://github.com/nagata1634/project-template.git
cp -r project-template/.github your-new-repo/
cp -r project-template/docs your-new-repo/
cp project-template/CLAUDE.md project-template/.markdownlint-cli2.yaml your-new-repo/
```

### b) degit で展開

```bash
npx degit nagata1634/project-template your-new-repo
cd your-new-repo
git init && git add . && git commit -m "chore: initial template"
```

## プレースホルダ一覧

コピー後、以下のプレースホルダを各リポジトリの値に置換する：

| プレースホルダ | 例 | 説明 |
|---|---|---|
| `{{ORG_NAME}}` | `acme-corp` | GitHub Organization 名（個人 user の場合は user 名） |
| `{{REPO_NAME}}` | `my-service` | リポジトリ名 |

一括置換例：

```bash
ORG=acme-corp
REPO=my-service
find . -type f \( -name '*.md' -o -name '*.yml' -o -name '*.yaml' \) \
  -exec sed -i.bak -e "s/{{ORG_NAME}}/$ORG/g" -e "s/{{REPO_NAME}}/$REPO/g" {} \;
find . -name '*.bak' -delete
```

## 初期セットアップ（コピー後の手順）

1. プレースホルダを置換
2. 初回 PR で `.github/workflows/` 3 種と `docs/MAINTENANCE.md` を投入
3. マージ後、Actions から「Setup branch protection (main)」を 1 回実行
4. Organization Ruleset を使う場合は `docs/ORG_RULESET.md` を参照

詳細は `docs/MAINTENANCE.md` を参照。

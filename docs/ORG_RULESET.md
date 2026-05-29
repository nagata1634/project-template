# Organization Ruleset 手順

組織配下の全リポジトリの `main` ブランチを一括保護するための Organization-level Ruleset 設定手順。BRANCHING.md §4 の補足として利用。

## なぜ Organization Ruleset か

| | 個別リポジトリ Branch Protection | **Organization Ruleset** |
|---|---|---|
| 設定箇所 | リポジトリそれぞれ | **1ヶ所** |
| 一貫性 | ズレる可能性あり | 保証される |
| テストモード | なし | **Evaluate** で事前検証可 |
| 新リポ適用 | 手動 | 自動（pattern マッチ） |
| Bypass リスト | あり | あり |

## 手順

### 1. ルールセット作成画面へ

`https://github.com/organizations/{{ORG_NAME}}/settings/rules`

「New ruleset」 → 「New branch ruleset」

### 2. 基本設定

**Ruleset name**: `main-protection-all-repos`

**Enforcement status**:
- 初期は `Evaluate`（チェックは走るがブロックしない）
- 評価期間中に Insights で 「そのルールだとどうなってたか」を見て、問題なければ `Active` に変更

**Bypass list**: (個人運用フェーズ) **空**。例外を作らない。

### 3. Target repositories

**Selected** を選び、対象リポジトリをチェックする。

> 例: `overview-repo`, `service-a`, `service-b` 等、組織配下で main 保護を適用したい全リポジトリ。テンプレート利用者は自組織のリポジトリ名に書き換えること。

または **All repositories** を選び、今後追加されるリポジトリも自動適用する。

### 4. Target branches

**Include default branch** をチェック。これで各リポジトリの default branch (`main`) が対象になる。

### 5. Branch rules

以下を **チェック**：

| ルール | 詳細 |
|---|---|
| **Restrict deletions** | main を削除不可 |
| **Block force pushes** | force push 不可 |
| **Require linear history** | merge commit 不可、squash または rebase のみ |
| **Require a pull request before merging** | ☑ |
| → Required approvals | `0` 程度（Reviewer フェーズはセルフ） |
| → Dismiss stale pull request approvals when new commits are pushed | ☑ |
| → Require review from Code Owners | ✖（CODEOWNERS 未設定のため） |
| **Require status checks to pass** | ☑ |
| → Required check | `typecheck-test` 等のジョブ名（CI 初回実行後に選択肢に出てくる） |
| → Require branches to be up to date before merging | ☑ |

以下は **チェックしない**：

- Restrict creations（ブランチ作成は自由）
- Restrict updates（作業ブランチの更新は自由）
- Require signed commits（署名必須は身丈にあわない）

### 6. Save

下の 「Create」 ボタンで保存。

### 7. 動作確認（Evaluate モード中）

1. 適用後、どこかのリポジトリでちょっとしたコミットを main へ直 push → 通ってしまうが記録に残る
2. `https://github.com/organizations/{{ORG_NAME}}/settings/rules` → ルールセット詳細 → **Insights** タブ で「ブロックされるはずだったイベント」を見る
3. 見た上で問題なければ Enforcement status を `Active` へ変更

## 以降の作業フロー（Active 化後）

1. 作業ブランチを切る `git checkout -b feat/your-change` または GitHub MCP の `create_branch`
2. コミット・push → PR 作成
3. CI green 待ち
4. Squash merge → main に入る
5. タグ `vX.Y.Z` push でデプロイ起動

**main は常に「デプロイ可能な状態」**。タグを打てばそのまま本番へ。

## Bypass / 緊急例外

- 本当に必要なときは Enforcement status を `Evaluate` に一時的に戻して作業 → `Active` へ戻す
- Bypass list にアカウントを追加するより Evaluate トグルの方が安全 (記録に残る)

## Insights / モニタリング

- `https://github.com/organizations/{{ORG_NAME}}/settings/rules` → 該当ルールセット → **Insights** タブ
- ブロック / Bypass / Pass の件数を追跡
- 規則が厳しすぎるケースもここで見える

## 関連

- [BRANCHING.md](BRANCHING.md) §4
- GitHub 公式ドキュメント: https://docs.github.com/en/organizations/managing-organization-settings/managing-rulesets-for-repositories-in-your-organization

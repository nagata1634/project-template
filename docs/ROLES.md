# ROLES — 7 ロール開発プロセス

全リポジトリに適用する開発ロールと、各フェーズの開始・完了条件、成果物。

```
Analyst → Architect → Implementer → Tester → Reviewer → Documenter → SecurityAuditor
```

| ロール | 目的 | 主要成果物 |
|---|---|---|
| Analyst | Why / 何を作るのかを明確化 | `docs/REQUIREMENTS.md` |
| Architect | どう作るかを設計 | `docs/ARCHITECTURE.md`, `docs/DECISIONS.md` |
| Implementer | コード化・デプロイ可能な状態 | `src/`, `tests/`, ランタイム設定ファイル (`wrangler.jsonc` / `Dockerfile` 等) |
| Tester | 自動・手動テスト | `docs/TEST_PLAN.md`, `tests/*` |
| Reviewer | セルフ品質チェック | `docs/SELF_REVIEW.md` |
| Documenter | 使い方と履歴 | `README.md`, `docs/CHANGELOG.md` |
| SecurityAuditor | 脅威モデル + 検証 | `docs/SECURITY_CHECK.md` |

## Analyst フェーズ

### 開始条件
「これを作ろう」と決めた段階。

### 完了条件 (Done Criteria)
- [ ] `docs/REQUIREMENTS.md` に以下が含まれる:
  - [ ] **Why** (目的・背景)
  - [ ] **ユースケース** 最低 3 件
  - [ ] **公開機能 / API 一覧 (v1)** 名前 + 用途
  - [ ] **データストア** (KV/R2/D1/JSONL/RDB 等のいずれか)
  - [ ] **認証・セキュリティ層** (L1〜LN)
  - [ ] **制約・前提**
  - [ ] **v1 スコープ外 (Won't Do)**
  - [ ] **やめる条件**
  - [ ] **自動デプロイ品質基準**
- [ ] 上位 REQUIREMENTS（組織共通要件）へのリンクあり（該当する場合）
- [ ] `requirements` ラベル付き issue で追跡

## Architect フェーズ

### 開始条件
Analyst 完了 (REQUIREMENTS がある)。

### 完了条件
- [ ] `docs/ARCHITECTURE.md`:
  - [ ] モジュール構成図
  - [ ] データモデル (KV キー設計、D1/RDB スキーマ、オブジェクトレイアウト等)
  - [ ] API 設計 (エンドポイント・シーケンス)
  - [ ] エラー設計 (error_code 一覧)
  - [ ] セキュリティ境界
- [ ] `docs/DECISIONS.md` 主要 ADR (Context / Decision / Consequences 形式)
  - [ ] 最低 3 件 ADR (ストレージ選定・認証方式・主要 trade-off)
- [ ] Implementer 着手可能と明白な概要 (「どこから手を付けるか」わかる)

## Implementer フェーズ

### 開始条件
REQUIREMENTS + ARCHITECTURE + DECISIONS が揃っている。

### 完了条件
- [ ] `tsc --noEmit` pass
- [ ] 主要関数が `src/` にある
- [ ] ランタイム設定ファイル (`wrangler.jsonc` / `Dockerfile` 等) にバインディング・設定反映済み
- [ ] 外部リソース (KV / R2 / D1 / バケット / DB 等) 作成 + ID 反映済み
- [ ] 最低限動作確認: 1 機能をエンドツーエンドで実行できる (curl / クライアント / テストランナー等)

## Tester フェーズ

### 開始条件
主要関数が実装されている。

### 完了条件
- [ ] `docs/TEST_PLAN.md` に `AT-XXX:` 形式で自動テストリスト
- [ ] `tests/` に対応する `.test.ts` ファイル
- [ ] **書き込み系機能は必ず単体テスト** (正常 + 異常)
- [ ] `vitest run` で all green
- [ ] CI で自動実行されることを確認

## Reviewer フェーズ

### 開始条件
Tester 完了。

### 完了条件
- [ ] `docs/SELF_REVIEW.md` に以下:
  - [ ] 設計上の懸念・トレードオフ
  - [ ] 改善案 (将来の TODO)
  - [ ] 既知の制約 / 意図的にやらないこと
- [ ] `docs/CODING_STANDARDS.md` とチェック済み
- [ ] デッドコード (使われていない export) の確認

## Documenter フェーズ

### 開始条件
Reviewer 完了。

### 完了条件
- [ ] `README.md` に what / why / how to use
- [ ] `docs/CHANGELOG.md` にバージョンごとの変更 (Keep a Changelog 風)
- [ ] `CONTRIBUTING.md` (BRANCHING.md へのリンク)
- [ ] スクリーンショットや例があるとよりよい (任意)

## SecurityAuditor フェーズ

### 開始条件
Reviewer 完了 (Implementer フェーズと並行可)。

### 完了条件
- [ ] `docs/SECURITY_CHECK.md` に `Sxx:` 形式の検証項目
- [ ] `docs/SECURITY_BASELINE.md` の S1〜S10 を全てチェック (済み / 対象外 / 不適用)。invite-only マルチユーザー対応は S11〜S12 も対象
- [ ] 脅威モデル (STRIDE 判断)、主要 3 件以上
- [ ] 残存リスクと緩和策を記載

## デプロイ条件 (全フェーズ揃って初めて)

以下すべてが満たされて初めて `vX.Y.Z` タグを打つ:

- [ ] 全 7 フェーズの完了条件がチェック済み
- [ ] REQUIREMENTS の合格条件を満たす
- [ ] CI green
- [ ] `docs/CHANGELOG.md` に該当バージョンエントリあり

## フェーズ間連携

- **並行可能**: Tester と SecurityAuditor (Implementer 後)
- **フィードバックループ**: Reviewer / SecurityAuditor が重大問題検出 → Architect / Implementer へ戻す
- **Documenter は最後だけど常時並行**: 各フェーズでドキュメントを書いておく

## Claude とロールの対応

Claude をロールごとに起動するコマンド例:

| ロール | プロンプト例 |
|---|---|
| Analyst | 「<対象> の要件を整理して docs/REQUIREMENTS.md に書いて」 |
| Architect | 「<対象> の ARCHITECTURE と DECISIONS を REQUIREMENTS から起こして」 |
| Implementer | 「<対象> の Implementer フェーズを進めて、src/ とランタイム設定を作って」 |
| Tester | 「<対象> の tests/ を書いて TEST_PLAN.md も更新」 |
| Reviewer | 「<対象> を CODING_STANDARDS と照らして SELF_REVIEW.md を作成」 |
| Documenter | 「<対象> の README と CHANGELOG を今の状態に揃えて」 |
| SecurityAuditor | 「<対象> に SECURITY_BASELINE を適用して SECURITY_CHECK.md を作る」 |

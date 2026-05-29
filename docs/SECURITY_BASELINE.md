# SECURITY BASELINE

全リポジトリに適用する最低限セキュリティ基準。SecurityAuditor フェーズで項目ごとにチェックし、`docs/SECURITY_CHECK.md` に記録する。

## S1. Secrets 管理

| | Workers | コンテナ (Docker 等) |
|---|---|---|
| 保存先 | `wrangler secret` | Docker secrets / env (image に焼き込まない) |
| 名前規則 | `AUTH_PASSWORD`, `GITHUB_TOKEN`, `DOCKER_TOKEN` etc. | 同上 |
| ローテーション | secret 上書き (年次 + 漏洩時) | 同上 |

**NG**
- コード / 設定ファイル / コミットメッセージに平文記載
- `vars` (Plain Text Var) に置く → ダッシュボードで見えてしまう

## S2. 認証

| 層 | 推奨方式 |
|---|---|
| クライアント ↔ Worker (公開エンドポイント) | **OAuth 2.1** (`@cloudflare/workers-oauth-provider` 等) |
| Worker ↔ コンテナ (内部 RPC) | Bearer (例: `DOCKER_TOKEN`) |
| 他サービス連携 (GitHub PAT 等) | サービス固有の token |

- `AUTH_PASSWORD` は最低 **32 文字**、特に厳重に保護したいシステムは **64 文字**
- パスワード比較は必ず `timingSafeEqual` (== 禁止)
- OAuth login form 等の HTML レスポンスでは HTML escape 必須

## S3. レート制限

プロジェクトの用途に応じて妖当な上限を定める。例：

| 用途 | 上限の目安 |
|---|---|
| 高頻度想定の汎用 API | 60 req/min/token |
| 中頻度のバックエンド API | 10 req/min/token |
| 低頻度想定の特殊用途 (管理系等) | 5 req/min/token |

- 実装: KV カウンタ `rl:{tokenHash}:{minute}` (TTL 60) 等
- 超過時は 429 Too Many Requests 、`RATE_LIMITED` error_code

## S4. 入力検証

- すべての公開 API / ツール引数を zod 等で validate
- ファイルパス系: ホワイトリスト + 親ディレクトリ脱出 (`..`) 禁止
- 文字数上限を明示 (body 10KB、namespace 名 100 文字等)
- タグ名 / namespace に `:` 不可 (キー区切り文字)

## S5. 出力サニタイズ

- HTML レスポンス (OAuth login 画面等): `escapeHtml()` で `& < > " '`
- JSON: JSON.stringify は安全だがユーザー入力をそのまま echo しない
- エラーメッセージに token / password / SSH key を含めない

## S6. 監査ログ

- **全書き込み操作を記録**
- マスキング必須キー: `token`, `secret`, `password`, `auth`, `github_token`
- Workers: KV `audit:{ulid}` (TTL なし、削除は手動のみ)
- コンテナ: JSONL append-only、ファイル削除不可
- スキーマ: `{ id, timestamp, operation, result, metadata }`

## S7. ネットワーク境界

### Workers
- `global_fetch_strictly_public` compatibility_flag (SSRF 対策)
- 外部 API 呼び出しはホワイトリストに限定

### コンテナ (オンプレ / NAS / セルフホスト)
- **Inbound: outbound のみ** (Worker からロングポーリング等)
- Outbound: コンテナ内 iptables で egress allowlist
- 例: 管理系コンテナはインターネット egress **完全遮断**、LAN 内 SSH/WinRM のみ
- 例: 外部サービス連携コンテナは該当ドメインのみ allowlist

## S8. コンテナ実行オプション (コンテナデプロイのみ)

- `--read-only`
- `--cap-drop=ALL` + 必要最低限 `--cap-add`
- 非 root user (`USER` directive in Dockerfile)
- `--security-opt=no-new-privileges`
- ボリュームマウントは書き込み必要な特定 path のみ

## S9. ブランチ保護

- main 直 push 不可
- PR 必須
- CI green 必須
- linear history (squash merge)
- force push / 削除禁止
- `main` / `master` / `develop` は delete_branch で保護 (GitHub MCP / Ruleset)

## S10. 依存管理

- Dependabot weekly (ドキュメント中心リポは monthly でも可)
- `dependencies` ラベル自動付与
- セキュリティ更新は即マージ検討
- メジャーバージョンアップは `breaking-change` ラベルで慎重に
- `npm audit` を CI で実行 (推奨)

## S11. アカウント管理 (invite-only マルチユーザー対応プロジェクトのみ)

該当プロジェクトのみ適用。public self-signup は別途要件で許可されない限り禁止。

| 項目 | 要求事項 |
|---|---|
| パスワードハッシュ | **PBKDF2-SHA256, iterations ≥ 600,000, salt 16 bytes 以上** (Web Crypto API)。bcrypt/Argon2 は Workers ランタイムで使用不可 |
| パスワード強度 | 最低 12 文字、英数字混在 |
| パスワード比較 | `timingSafeEqual` 必須（== 禁止） |
| ログイン lockout | 5 回連続失敗で 15 分 lockout (`lockout:{email}:{epochMinute}` カウンタ) |
| セッション | OAuthProvider の access token 経由のみ。直接 cookie session は不使用 |
| パスワード変更 | 現パスワード入力必須 |
| 監査ログ | ログイン成功・失敗・lockout・パスワード変更を必ず記録 (S6 と整合) |

**NG**
- パスワードを平文 / base64 / SHA-256 単純ハッシュで保存
- 失敗回数制限なしのログイン (brute force)
- 同一 IP からの無制限アカウント作成 (招待 token 単発消費 + per-IP rate limit で防御)

## S12. 招待 token (invite-only マルチユーザー対応プロジェクトのみ)

| 項目 | 要求事項 |
|---|---|
| 形式 | ULID (時系列ソート可、衝突困難) |
| TTL | 72 時間 (KV `expirationTtl`) |
| 消費 | **単発**。アカウント作成成功時に KV から削除 |
| 失効 | admin が `admin_invite_revoke` で即時無効化可能 |
| email バインド | 招待時に admin が email を指定。ユーザーは指定された email でのみ登録可（誤配送・乱用防止）|
| 発行レート制限 | admin あたり 10 招待/時間 |

**NG**
- 再利用可能な招待 token
- email 指定なしの open invite (実質 self-signup と等価)
- TTL 無制限の招待 token

## SecurityAuditor テンプレート

各プロジェクトの `docs/SECURITY_CHECK.md` に以下形式で記載:

```markdown
# SECURITY CHECK — <project-name>

> SECURITY_BASELINE.md の S1〜S10 (+ invite-only プロジェクトは S11〜S12) をチェック。追加のプロジェクト固有事項も記載。

## チェック一覧

| ID | 項目 | 状態 | 備考 |
|---|---|---|---|
| S1 | Secrets 管理 (vars に置かない) | ✅ | wrangler secret 使用 |
| S2 | 認証方式 | ✅ | OAuth 2.1 |
| S3 | レート制限 | ✅ | 60 req/min/token |
| S4 | 入力検証 (zod) | ✅ | 全エンドポイントで実装 |
| S5 | 出力サニタイズ | ✅ | escapeHtml あり |
| S6 | 監査ログ | ⚠️ | mask 未実装 → P1 修正 |
| S7 | ネットワーク境界 | ✅ | SSRF flag あり |
| S8 | コンテナ実行オプション | N/A | Workers のため不要 |
| S9 | ブランチ保護 | ✅ | main protected |
| S10 | 依存管理 | ✅ | Dependabot 設定済み |
| S11 | アカウント管理 | N/A | 単一認証のため対象外 |
| S12 | 招待 token | N/A | 同上 |

## 脅威モデル (STRIDE)

| 脅威 | 該当 | 緩和策 |
|---|---|---|
| Spoofing | 想定外クライアントからのアクセス | OAuth + AUTH_PASSWORD |
| Tampering | コード改ざん | branch protection + tag immutable |
| Repudiation | 操作を後で否認 | 監査ログ削除不可 |
| Information Disclosure | secret 漏洩 | mask + scope 最小化 |
| Denial of Service | rate spam | rate limit + DDoS 対策 |
| Elevation of Privilege | container 脱走 | --read-only + cap drop |

## 残存リスク

- (例) インフラ障害時にサービス全体停止 → 受け入れるリスク、代替手段なし
- ...
```

## 関連

- [`ROLES.md`](ROLES.md) SecurityAuditor フェーズ詳細
- [`CODING_STANDARDS.md`](CODING_STANDARDS.md) 実装レベルの規約

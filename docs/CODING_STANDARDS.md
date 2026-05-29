# CODING STANDARDS — 共通規約

全リポジトリ共通の TypeScript / コミット / ファイル構成 / テスト規約。Reviewer フェーズでチェックに使う。

## TypeScript

### tsconfig
- `strict: true` 必須
- `noUncheckedIndexedAccess: true` 推奨
- `noEmit: true` (ビルドはランタイム側 (wrangler / docker build 等) に任せる構成が前提)

### 型
- **`any` 禁止**。代わりに `unknown` + type narrowing
- `interface` を `type` より優先 (拡張性)
- public 関数は引数と戻り値の型を明示
- type-only は `import type {...}` で import
- オプショナルチェーン `obj?.prop?.method?.()` を活用

### 命名

| 対象 | 規則 | 例 |
|---|---|---|
| ファイル | kebab-case | `auth-handler.ts` |
| ディレクトリ | kebab-case | `src/`, `tests/`, `worker/`, `docker/` |
| 関数・変数 | camelCase | `memoryCreate`, `byteLength` |
| interface / type / class | PascalCase | `MemoryRecord`, `ServiceError` |
| 定数 | SCREAMING_SNAKE | `MAX_BODY_BYTES`, `RATE_LIMIT_PER_MIN` |
| RPC / ツール / エンドポイント | `{namespace}_{action}` | 例: `memory_create`, `linux_run_git` |
| error_code | SCREAMING_SNAKE | `UNAUTHORIZED`, `RATE_LIMITED` |

### エラー
- 共通エラー型 (例: `AppError`, `McpError` 等プロジェクトの基底型) を使う
- error_code は SCREAMING_SNAKE_CASE
- `catch (e: unknown)` で型を狭める
- クライアントへは `error_code` + `message`、スタックトレースは返さない

### 非同期
- Promise はちゃんと await
- top-level `void` で fire-and-forget 禁止 (Worker なら `ctx.waitUntil()` を使う)
- 並列可能なところは `Promise.all(...)`

### import
- 相対 import は `./xxx` 形式、深い `../../../` 禁止。`shared/` 経由で解決
- node 組込みは `node:fs` と prefix つき

## ファイル構成

以下は典型例。プロジェクトの形態に応じて取捨選択する。

### 例 1: 単一 Worker 完結型
```
.
├── src/
│   ├── index.ts        # エントリポイント
│   ├── types.ts        # Env, 共通 interface
│   ├── errors.ts       # AppError + ErrorCode
│   ├── auth-handler.ts # 認証 / OAuth UI 等
│   ├── server.ts       # ルーティング / ハンドラ登録
│   ├── store.ts        # ドメインロジック (KV/R2/D1)
│   └── lib/            # ユーティリティ
├── tests/
│   ├── _mock-*.ts      # モック
│   └── *.test.ts
├── docs/
├── wrangler.jsonc      # (Cloudflare Workers の場合)
├── tsconfig.json
└── package.json
```

### 例 2: Worker + コンテナ型（npm workspaces）
```
.
├── worker/         # エッジ Worker（gateway）
├── docker/         # コンテナワーカー
├── shared/         # 型定義 (worker/docker 共通)
├── docs/
└── package.json    # npm workspaces ルート
```

## テスト

### vitest
- すべての public 関数に最低 1 テスト
- 書き込み系は正常 + 異常両方
- インメモリモックは `tests/_mock-*.ts` に集約
- `makeEnv()` は毎テストで新規インスタンスを返す (テスト間で state を漏らさない)

### test 命名
```typescript
describe("memoryCreate", () => {
  it("AT-008: creates a memory and writes mem + tag index keys", ...);
  it("AT-009: throws PAYLOAD_TOO_LARGE for body > 10KB", ...);
});
```
- `AT-XXX:` は `docs/TEST_PLAN.md` の ID と対応
- it 本文は「何ができる/起きる」を短文で

### カバレッジ目安
- store / auth 系ロジック: 90%+
- index / boilerplate: 50%+ でも可 (手動テストで補う)

## コメント

### 書く時
- **Why** (なぜ) を書く。**What** (何) は code が語る
- 非自明な制約・workaround・歴史的背景
- 公開 API には JSDoc (`@param`, `@returns`)

### 書かない時
- 自明な処理に冗長コメント
- TODO は issue 化 (コードに残さない)
- 「今回の修正」を説明するコメント (タスク名や PR 番号を含めるやつ)

## ログ

- Workers: `console.log` / `console.error` で十分、Cloudflare Logs で収集
- **機密 (token/password) をログに出さない**、マスキング必須
- error path は詳細、success path は簡潔に
- 構造化ログは JSON.stringify して 1 行で出す

## 依存パッケージ

- 追加前に「本当に必要か」を検討
- メジャーバージョンアップは breaking-change ラベルで慎重に
- security update は即マージ検討
- Dependabot 設定 (`dependencies` ラベル自動付与) に従う

## スタイル・フォーマッタ

- prettier を使わないが、以下を手動で守る:
  - 2 スペース indent
  - ダブルクオート `"`
  - 末尾セミコロン含む
  - 100 文字以内 推奨

## Reviewer 用チェックリスト

Reviewer フェーズで以下をざっと見る:

- [ ] `any` 使っていないか
- [ ] error_code を一貫して使っているか
- [ ] テストが書き込み系をカバーしているか
- [ ] デッドコード / 未使用 import がないか
- [ ] Secrets をログ・エラーメッセージに含めていないか
- [ ] 不適切な `as any` / `// @ts-ignore` がないか
- [ ] tests/ を含む tsc が green
- [ ] SECURITY_BASELINE.md を議論した (詳細は SecurityAuditor)

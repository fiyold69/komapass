# 技術構成と開発環境

## 技術構成

| 層 | 使うもの | 決めた日 |
| --- | --- | --- |
| フロント | React＋Vite＋TypeScript、React Router。サーバー状態は最初は自前のfetch、フェーズ3でTanStack Query | 2026-10-11 |
| API | Hono（Node.js上）。フロントとは型をHonoのRPCで共有 | 2026-10-11 |
| DB | MySQL（手元はDocker）＋Drizzle ORM（マイグレーション込み） | 2026-10-12（[db.md](db.md)） |
| 認証 | Better Auth。MVPは全員LINEログイン。メールとパスワードはMVPの後（#23） | 2026-10-12、LINEだけにしたのは 2026-10-15（[auth.md](auth.md)） |
| 検証・テスト | zod、Vitest | 2026-10-11 |
| 公開先 | Railway Hobby | 2026-10-12（[deploy.md](deploy.md)） |

使わないもの：Next.js、Supabase、Cloudflare Workers、Bun（MVPの後に検討するものを含む）。

### 型の共有：Hono RPC（OpenAPIは後で）

- 決定：MVPはHonoのRPCでフロントとAPIの型を共有する。OpenAPIは、APIを別の言語で書き直すとき（Go）や外部に公開するときに足す
- 理由：1人で、フロントもAPIもTypeScriptなら、RPCで十分。追加のファイルが要らない
- 注意（Hono公式）：両方の `tsconfig` を `strict: true` にし、Honoの版をそろえる。`c.notFound()` は型が取れないので `c.json(…, 404)` を使う
- 決めた日：2026-10-11

### APIの3層

- `routes/`：HTTPの受け口。入力をzodで検証し、結果をステータスコード付きの `c.json()` で返す
- `services/`：業務ルール。HTTPもSQLも出てこない（DBなしで単体テストできる）
- `db/`：Drizzleのスキーマと問い合わせ

## 開発環境

### WSL2（Ubuntu）

- 決定：Windows 11のWSL2（Ubuntu）で開発する。コードは `~/` 以下に置き、`/mnt/c` には置かない。エディタはWindowsのVS Codeから拡張機能「WSL」でつなぐ
- 理由（Docker公式など）：
  - Windows側に置いたファイルをコンテナから読むと大幅に遅くなる
  - ファイル変更の通知がLinux側のファイルでしか届かず、Viteなどの自動リロードが効かなくなる
  - make、シェルスクリプト、改行コード（LF）が、デプロイ先やGitHub ActionsのLinuxとそろう（一般論）
- 決めた日：2026-10-11

### Node.jsとDocker

- 決定：フェーズ0で入れる（設計週では入れない）
- 決めた日：2026-10-12

### gitの設定

- `user.email` はGitHubの `…@users.noreply.github.com` のアドレスにする（本物のメールを公開リポジトリの履歴に残さないため）
- 既定のブランチ名は `main`

## おすすめ（未決。[open-questions.md](open-questions.md)）

| 対象 | おすすめ | 理由 |
| --- | --- | --- |
| JSのパッケージ管理 | pnpm | 依存を1か所に保存して共有し、宣言していない依存を使えない構造で事故を防ぐ（公式）。ワークスペースに対応 |
| Lintと整形 | Biome | 1つの道具と1つの設定（`biome.json`）で済む。ESLintのプラグインにしかないルールが要ったら、その時にESLintを足す |
| コマンドの入口 | Makefile | 準備は30分〜1時間。`up / down / migrate / seed / test / set-role / 時限表の登録（名前は未定）` を1か所にまとめられる |
| フォルダ | `apps/web`・`apps/api` の2つ | 準備は1〜2時間。`packages/shared` は必要になってから作る |

どれを選んでも、lockファイルをコミットし、使う版を `package.json` の `packageManager` 欄で固定する（固定方法の詳細は要確認）。

## 決定の履歴

| 日付 | 内容 | 関連 |
| --- | --- | --- |
| 2026-10-11 | 技術構成（React・Hono・zod・Vitest）、Hono RPC、WSL2 | — |
| 2026-10-12 | DBをMySQLに、公開先をRailwayに。Node.jsとDockerはフェーズ0で入れる | — |

## 出典

- [WSL 2 best practices for Docker Desktop on Windows](https://docs.docker.com/desktop/features/wsl/best-practices/)
- [pnpm: Motivation](https://pnpm.io/motivation)
- [Biome](https://biomejs.dev/)
- [Hono: RPC](https://hono.dev/docs/guides/rpc)

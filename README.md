# KomaPass（仮）

個別指導塾の講師が「出られないコマ」の代講を募集し、ほかの講師が申し出て、依頼した講師が1人を選んで成立させるWebアプリ。

> 現在は設計を終え、開発の土台づくり（フェーズ0）に入ったところ。

## できること（MVP）

- 講師：代講の依頼を出す・申し出る・取り下げる・申し出た人から選んで成立させる・取り消す
- 教室長：成立した代講の一覧を見る、新しく登録した人を承認する
- ログインはLINE

## 技術構成

React・Vite・TypeScript／Hono（Node.js）／MySQL・Drizzle ORM／Better Auth（LINEログイン）／zod・Vitest／Railway

## 動かし方

（準備中）

## ドキュメント

| 文書 | 中身 |
| --- | --- |
| [docs/decisions/](docs/decisions/) | 今の決定と理由、未決事項、悩んだこと |
| [docs/user-stories.md](docs/user-stories.md) | ユーザーストーリーと完了条件 |
| [docs/concurrency-cases.md](docs/concurrency-cases.md) | 同時操作のケースと、MVPでどこまで守るか |

## 進め方

- 1つのIssueを1つのブランチとPRで片付け、Squash mergeする
- AIは、相談・レビュー・調べものに使い、コードは自分で書く。AIに頼んだことは、PRに記録している
- 学習の記録は docs/retro/ に置く（予定）

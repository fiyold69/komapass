# 公開先と運用

## 決まっていること

### 公開先

- 決定：Railway Hobby（月5ドル、自費）。APIとMySQLを同じ場所に置き、画面と `/api` を同じドメインで配信する
- 理由：完成後に塾で実際に使ってもらうので、「止まらない」「データが消えない」を優先する。無料枠は、使われないと止まったり、期限で消えたりするため実運用に向かない。同じドメインにすると、Cookie（ログイン状態）の扱いが単純になる
- 決めた日：2026-10-12
- メモ：空のアプリを出す段階では、Railwayの新規アカウントに付く1回きりのクレジットで足りる見込み（不確か。使う前に公式の料金ページで確認）。本番公開の時点でHobbyにする

### 運用

- 決定：完成後、塾で実際に使ってもらう
- 理由：ポートフォリオとしても、実際に使われている方が強い
- 決めた日：2026-10-12
- 前提：塾の了承を記録に残る形でもらう（[security.md](security.md)）。バックアップの手順と障害に気づく方法は未決（[open-questions.md](open-questions.md)）

### MVPの後の発展課題

- Google Cloud（Cloud Run）や Cloudflare（Workers）への移行は、MVPの後の発展課題にする
- アプリをDockerイメージにしておけば、移行は「同じものを別の場所で動かす」作業になる

## 「止まる」の3種類（10月12日の整理）

| 種類 | 何が起きるか | 例（10月12日時点） |
|---|---|---|
| ① 使われないと止まる | 一定時間アクセスがないと止まり、次のアクセスで起動を待つ | Render無料のAPIは15分で停止 |
| ② 期限切れ | 決まった日数で使えなくなる | Render無料のPostgresは30日 |
| ③ 障害・メンテナンス | 運営側の都合で一時的に止まる | 有料でも起こりうるが、まれ |

塾で使うと①と②が致命的になる。有料のRailway Hobbyなら①と②はなく、③だけを気にすればよい。

## 比べた候補（10月12日時点の調査）

料金と無料枠は変わるので、使う前に公式で確認する。

| 候補 | 判断 | 理由 |
|---|---|---|
| Railway Hobby | **採用** | 月5ドル。APIとMySQLを同じ場所に置け、手間が少ない |
| Google Cloud | MVPの後 | Cloud Runには常に無料の枠があるが、Cloud SQLにはなく、DB代が主になる |
| Cloudflare Workers | MVPの後 | 無料プランはCPU時間の制限が厳しく、外部DBにつなぐ仕組み（Hyperdrive）も要る。手元での動かし方も違う |
| AWS | 見送り | 新規の無料プランはクレジット制で、期間が過ぎると有料。設定（VPC・IAM）が多い |
| Render＋Aiven（無料） | 不採用 | 0円だが、上の①②があり実運用に向かない |

## 決定の履歴

| 日付 | 内容 | 関連 |
|---|---|---|
| 2026-10-12 | Railway Hobbyに決定。実運用を前提にする | — |

## 出典

- [Railway: Pricing Plans](https://docs.railway.com/pricing/plans)
- [Render: Deploy for Free](https://render.com/docs/free)
- [Aiven for MySQL free tier](https://aiven.io/docs/products/mysql/concepts/mysql-free-tier)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Google Cloud: Free cloud features and trial offer](https://docs.cloud.google.com/free)
- [Cloudflare Workers: Limits](https://developers.cloudflare.com/workers/platform/limits/)

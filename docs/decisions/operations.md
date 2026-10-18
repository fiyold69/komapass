# 公開先と運用

本番の置き場所、運用の前提、開発者が手で行う操作（スクリプト）の約束。

## 公開先

- Railway Hobby（月5ドル、自費）。APIとMySQLを同じ場所に置き、画面と `/api` を同じドメインで配信する
- 完成後に塾で実際に使ってもらうので（[product.md](product.md)）、「止まらない」「データが消えない」を優先するため。無料枠は、使われないと止まったり、期限で消えたりするため実運用に向かない
- 同じドメインにすると、Cookie（ログイン状態）の扱いが単純になるため
- 空のアプリを出す段階では、Railwayの新規アカウントに付く1回きりのクレジットで足りる見込み（不確か。使う前に公式の料金ページで確認）。本番公開の時点でHobbyにする

## 実運用の前提

- 塾の了承を、記録に残る形でもらう（[security.md](security.md)）
- バックアップの手順と障害に気づく方法は未決（[open-questions.md](open-questions.md) の「バックアップの手順、障害に気づく方法」）

## スクリプトの約束

役割の付与、最初の開発者の承認、時限表の登録など、開発者が手で行う操作に共通する約束。

- 本番DBにSQLを手で打たない。必ずスクリプト経由で書き換える
- 手元の開発用DBも、できるかぎりスクリプト経由で書き換える。直接SQLを打つのは最小限にする
- スクリプトの中で、入力の値が正しいか（例：その人が存在するか、役割の値が正しいか、時限の時刻が正しいか）をzodで確かめてから書き換える
- スクリプトはリポジトリ（`scripts/`）に残し、何をしたかの記録にする
- コマンドの入口は Makefile（[dev-env.md](dev-env.md)）

## 時限表の入れ方

- MVPでは、開発者がスクリプトで入れる
- 教室長が画面で入れる機能はMVPの後

## MVPの後の発展課題

- Google Cloud（Cloud Run）や Cloudflare（Workers）への移行は、MVPの後の発展課題にする
- アプリをDockerイメージにしておけば、移行は「同じものを別の場所で動かす」作業になる

## 「止まる」の3種類（2026-10-12 時点の整理）

| 種類 | 何が起きるか | 例（10月12日時点） |
| --- | --- | --- |
| ① 使われないと止まる | 一定時間アクセスがないと止まり、次のアクセスで起動を待つ | Render無料のAPIは15分で停止 |
| ② 期限切れ | 決まった日数で使えなくなる | Render無料のPostgresは30日 |
| ③ 障害・メンテナンス | 運営側の都合で一時的に止まる | 有料でも起こりうるが、まれ |

塾で使うと①と②が致命的になる。有料のRailway Hobbyなら①と②はなく、③だけを気にすればよい。

## 比べた候補（2026-10-12 時点の調査）

料金と無料枠は変わるので、使う前に公式で確認する。

| 候補 | 判断 | 理由 |
| --- | --- | --- |
| Railway Hobby | **採用** | 月5ドル。APIとMySQLを同じ場所に置け、手間が少ない |
| Google Cloud | MVPの後 | Cloud Runには常に無料の枠があるが、Cloud SQLにはなく、DB代が主になる |
| Cloudflare Workers | MVPの後 | 無料プランはCPU時間の制限が厳しく、外部DBにつなぐ仕組み（Hyperdrive）も要る。手元での動かし方も違う |
| AWS | 見送り | 新規の無料プランはクレジット制で、期間が過ぎると有料。設定（VPC・IAM）が多い |
| Render＋Aiven（無料） | 不採用 | 0円だが、上の①②があり実運用に向かない |

## 出典

- [Railway: Pricing Plans](https://docs.railway.com/pricing/plans)
- [Render: Deploy for Free](https://render.com/docs/free)
- [Aiven for MySQL free tier](https://aiven.io/docs/products/mysql/concepts/mysql-free-tier)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Google Cloud: Free cloud features and trial offer](https://docs.cloud.google.com/free)
- [Cloudflare Workers: Limits](https://developers.cloudflare.com/workers/platform/limits/)

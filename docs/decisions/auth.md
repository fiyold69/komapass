# 認証・アカウント・役割

ログインの方法、新しい人の扱い、役割の付け方。ライブラリは Better Auth。

## ログインの方法

- Better Auth を使う。MVPでは全員（講師・教室長・開発者）がLINEで入る。メールとパスワードのログインはMVPの後（#23）
- LINEログインがあれば、講師が新しいパスワードを覚えずに済み、実運用で使われやすいため。今の利用者は塾の講師・教室長と開発者1人だけで、全員がLINEを使える。作るものを減らし、MVPを早く使ってもらうため
- セッションとCookieの仕組みは、LINEログインを作るときに学ぶ

## 承認待ち方式

- 新しく登録した人は、教室長か開発者が承認するまで何も操作できない
- LINEアカウントは誰でも持っているので、そのままだと塾と関係のない人も入れてしまうため。承認待ちなら、承認画面と「承認待ち」の状態だけで済むため
- 承認を戻して利用を止める機能はMVPの後（#15）

## 役割と承認状態の持ち方

- Better Auth の `user` テーブルに列を足して持つ（`additionalFields`）。利用者が自分で書き換えられない設定（`input: false`）にする
- 役割がセッションの型にも付き、JOINが要らないため。1人1役割なので、列1つで足りる。複数教室に広げるときは、別テーブル（例：所属）へ移す
- 不確か：`input: false` の具体的な効き方は、作る前に Better Auth の公式ドキュメントで確認する

## 役割を付ける操作

- MVPでは画面もAPIも作らず、開発者が `make set-role` のようなスクリプトで付ける。画面から付ける機能はMVPの後
- 理由は3つ。①役割を付けるのは最初の1回と人が入れ替わるときだけで、画面・API・権限チェック・テストを作るのは重い。②「役割を変えるAPI」は権限を上げる危ない操作なので、作らないのが最も安全。③本番DBにSQLを手で打つより、スクリプトの中で確かめてから書き換える方が安全
- 最初の開発者は、まずLINEで入り（承認待ちになる）、そのあとスクリプトで「承認済み」と `superadmin` を付ける。環境変数で最初から特別扱いする方法や、「最初に登録した人を自動で開発者にする」方法は採らない。ログインの処理に特別扱いが入り、権限を上げる入口になるため
- スクリプトの約束は [operations.md](operations.md)
- 要確認：Better Auth の admin プラグインにも役割を変える機能がある。このプロジェクトは役割を `user` テーブルに自分で足した列で持つので、そのまま合うかは不確か。使う前に確認する

## LINEログインのメール

- LINEで入る人は、常に仮アドレス（LINEのユーザーIDから作る、`.invalid` で終わる届かないアドレス）を `user.email` に入れる。LINEからメールをもらえても使わない
- Better Auth は全ユーザーにメールを必要とするが、公式に仮アドレスの方法が案内されているため。常に仮アドレスにすると、同じメールの人との自動の結びつきが起きず、持つ個人情報も減るため
- メール確認を必須にする設定は、LINEログインに付けない（仮アドレスは確認メールを受け取れない）
- `user.email` の中身と、LINEの人を見分ける鍵は [db.md](db.md)
- 不確か：LINEのユーザーを見分ける項目名と、仮アドレスを入れる具体的な設定（`mapProfileToUser` など）。作る前に公式で確認する（下の「LINEログインの仮アドレス（調べたこと）」）

## メールの扱い

- メールの送受信はしない。メールはアカウントの識別とログインにだけ使う
- メール送信の仕組み（送信サービス、届かないときの対応）を持たずに済むため

## パスワードの再設定

- MVPではメールとパスワードのログインがないので、再設定もない。メールとパスワードのログインと一緒にMVPの後で決める（#23）
- Better Auth の再設定は、メールでリンクを送る仕組み

## LINEアカウントをなくした講師

- 古いアカウントを止め、新しいアカウントを承認する運用で対応する。記録の引き継ぎはMVPの後
- 引き継ぎの仕組みを作るより、承認待ち方式の運用で足りるため
- 利用を止める機能はMVPの後（#15）なので、MVPの間は起きたときに「スクリプトで止める」か「止めずに置いておく」かを判断する

## LINEログインの仮アドレス（調べたこと）

2026-10-15 時点で、公式ドキュメントで確かめたことと、確かめきれていないこと。作る前にもう一度、公式で確認する。

確かめたこと：

- Better Auth は、すべてのユーザーにメールを必要とする。プロバイダーがメールを返さないと、`error=email_not_found` で失敗する（Better Auth: OAuth）
- その対策として、`mapProfileToUser` で「プロバイダーの変わらないID」から仮アドレスを作る方法が公式に案内されている。届かないドメイン（`.invalid`）を使うよう書かれている。公式の例は Discord・Apple・Microsoft で、LINE の例はない
- `mapProfileToUser` の戻り値は「利用者からの入力」として扱われる。`additionalFields` で `input: false` にした項目（役割・承認状態）は無視される。なので、LINEのプロフィールから役割や承認状態を書き換えられることはない
- `mapProfileToUser` で、アカウントの識別（`account` の「プロバイダー＋ID」）は変えられない
- 2回目以降のログインは、`account` の「プロバイダー＋ID」の組で見つける
- Better Auth の LINE の既定のスコープは `openid`・`profile`・`email`。`disableDefaultScope` で既定を外し、`scope` で必要なものだけにできる
- LINE の IDトークンには、必ず `sub`（ユーザーID）が入る。`email` は `email` スコープを付けたときだけ入る
- LINE のユーザーIDは、**プロバイダー（LINE Developers の「プロバイダー」）ごとに違う値**になる。同じ人でも、別のプロバイダーのチャネルでは別のIDになる（LINE: Getting user IDs）
- LINE からメールをもらうには、コンソールで「メールアドレスの取得権限」を申請する必要がある。利用者が同意画面で断ると `ACCESS_DENIED` になる

ここから考えられること（案。決定ではない）：

- メールを使わない決定なので、`email` スコープを外す（`disableDefaultScope` ＋ `scope: ["openid", "profile"]`）と、申請も個人情報も要らなくなる。既定のまま `email` を要求したとき、申請していないチャネルでどうなるかは不確か
- 仮アドレスは LINE の `sub` から作ることになる。Better Auth の LINE の `mapProfileToUser` に渡ってくるプロフィールの項目名（`sub` か別の名前か）は、公式ページに書かれていない（不確か。ソースコードか、実際に動かして確かめる）
- ユーザーIDはプロバイダーごとに変わるので、本番のLINEのチャネルは、最初に作ったプロバイダーから動かさない。移すと、全員が別人として扱われる

## LINEログインの準備（メモ）

- LINE Developers でLINEログインのチャネルを作り、チャネルIDとシークレットを受け取り、コールバックURLを登録する
- 作ったばかりのチャネルは「開発中」で、管理者とテスターに登録した人しかログインできない。誰でも使えるようにするには「公開」に切り替える（LINE公式）
- LINEからメールアドレスをもらうには申請が要る可能性がある。ただし、上の決定でメールは使わないので、申請はしない予定

## 出典

- [Better Auth: Database（additionalFields）](https://www.better-auth.com/docs/concepts/database)
- [Better Auth: Admin plugin](https://www.better-auth.com/docs/plugins/admin)
- [Better Auth: LINE](https://www.better-auth.com/docs/authentication/line)
- [LINE Developers Console overview](https://developers.line.biz/en/docs/line-developers-console/overview)
- [Better Auth: OAuth（メールを返さないプロバイダー、mapProfileToUser）](https://www.better-auth.com/docs/concepts/oauth)
- [LINE: Verify ID token（IDトークンの中身）](https://developers.line.biz/en/docs/line-login/verify-id-token/)
- [LINE: Integrating LINE Login（メールアドレスの取得権限）](https://developers.line.biz/en/docs/line-login/integrate-line-login/)
- [LINE: Getting user IDs（ユーザーIDはプロバイダーごと）](https://developers.line.biz/en/docs/messaging-api/getting-user-ids/)

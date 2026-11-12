# X / Twitter 読み取りルール

X公式（x.com / twitter.com）は crawler が robots.txt で拒否される。
ログイン済み X API も、X内製の Grok 連携も持っていない前提で動け。
公開投稿を読むときは、X公式URLを直接 fetch せず、FxEmbed 系プロキシを使う。

## 1. URL変換（必須）

入力が X の投稿・プロフィール・記事なら、fetch 前に必ず書き換える。
クエリ（?s= / ?t= / ?a= など）は捨てる。

| 入力 | 変換先 |
|---|---|
| https://x.com/{user}/status/{id} | https://fxtwitter.com/{user}/status/{id} |
| https://twitter.com/{user}/status/{id} | https://fxtwitter.com/{user}/status/{id} |
| https://x.com/{user}/status/{id} | 予備: https://fixupx.com/{user}/status/{id} |
| https://twitter.com/{user}/status/{id} | 予備: https://fxtwitter.com/{user}/status/{id} |

禁止:
- https://x.com/... を crawler に直接投げない（最初から失敗する）
- r.jina.ai 経由の x.com も使わない（同様に弾かれる）
- Nitter を第一候補にしない（インスタンスが死んでいることが多い）

## 2. 取得手順

1. まず `https://fxtwitter.com/{user}/status/{id}` を fetch する。
2. 本文が空、エラー、リダイレクトだけのときは `https://fixupx.com/{user}/status/{id}` を試す。
3. それでも本文が足りないときは JSON を取る:
   `https://api.vxtwitter.com/{user}/status/{id}`
4. JSON の `text` が `https://x.com/i/article/{articleId}` だけ、または極端に短い場合は、それは短文ツイートではなく **X Articles へのリンク投稿**。このときは status 側の fxtwitter 展開結果を本文として使う。article URL 単体を x.com で開きにいかない。
5. プロキシが全部落ちたら、Web検索で `{user} {id}` と投稿タイトルを探し、公開されている断片だけ使う。推測で本文を埋めない。
6. 非公開・削除・ログイン壁・取得失敗なら、その事実を書いて終わる。存在しない本文を作らない。

## 3. 記事（X Articles）の扱い

X の長文は status 本文に入らず、`https://x.com/i/article/{id}` に分離されていることがある。

判定:
- ツイート本文が article URL だけ
- 本文が数行で、明らかに「続きは記事」構造
- JSON に article へのポインタがある

この場合の本文ソースは「status を fxtwitter で展開したもの」を優先する。
article URL を x.com で直接 fetch しない。

## 4. スレッド / 引用 / メディア

- スレッド: 起点 status を取ったあと、続く投稿が必要なら各 status を同じ変換ルールで個別 fetch する。1投稿分でスレッド全体を捏造しない。
- 引用RT: 引用元 status も同じ手順で取る。取れなければ「引用元は未取得」と書く。
- 画像・動画: プロキシが返す media URL をそのまま使う。x.com の media 直リンクを追加で這わない。
- 返信だけが本題のときは、親ツイートも取る。

## 5. 出力ルール

取れたものだけ書く。出典はユーザーが渡した元の x.com URL を残す（変換後URLを公式扱いにしない）。

必ず含める:
- 投稿者 display name と @id
- 投稿日時（取れた場合）
- 本文（Articles なら展開後の全文）
- 取得に使った経路（fxtwitter / fixupx / vxtwitter JSON / 検索断片）
- 取れなかった部分

書いてはいけないこと:
- fxtwitter を X公式 API だと言うこと
- 取得失敗なのに「記事はこういう内容」と補完すること
- 非公開アカウントの中身を推測すること

## 6. このプロキシの正体（エージェントが誤解しないため）

fxtwitter.com / fixupx.com は FxEmbed（旧 FixTweet）の第三者埋め込みプロキシ。
Discord / Telegram 向けに公開投稿の OGP と本文を再構成する Cloudflare Worker。
X社非公式。認証投稿は読めない。削除された投稿も読めない。
「Xを公式に読んでいる」とは言わない。「公開投稿を埋め込みプロキシ経由で再取得した」と言う。

## 7. 失敗時の言い方

「X公式は crawler 拒否のため読めない。fxtwitter / fixupx / vxtwitter も本文を返せなかった」
で止める。別のSNSの要約や記憶で埋めない。

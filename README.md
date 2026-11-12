# x_twitter_reading — X（Twitter）読み取りスキル

AIエージェントに X（旧Twitter）の公開投稿を読ませるための Agent Skills 形式スキルです。

X公式（x.com / twitter.com）は crawler が robots.txt で拒否されるため、通常の Web fetch では公開投稿ですら取得できません。このスキルは、FxEmbed 系プロキシ（fxtwitter → fixupx → vxtwitter JSON）経由で公開投稿・スレッド・引用・X Articles を取得する手順と、取得できなかった場合に捏造しないための出力ルールをまとめたものです。

## 入っているもの

- `x_twitter_reading/SKILL.md` — 手順と出力ルールの正本
- `x_twitter_reading/skill.yaml` — スキルのメタ情報（名前・説明・カテゴリ）

認証情報や個人情報は含まれていません。

## 使い方（導入）

1. `x_twitter_reading` フォルダごと、自分のエージェント環境のカスタムスキル置き場へコピーする
2. エージェントに X の投稿 URL（x.com / twitter.com）を渡して「読んで」と頼む
3. スキルが発動し、公式URLを直接 fetch せずプロキシ経由で本文を取得する

スキル形式に対応した環境（Muse のカスタムスキル、Claude Code など）で使えます。

## 注意

- 読めるのは**公開投稿だけ**です。非公開アカウント・削除済み投稿・ログイン壁の先は読めません
- 冒頭の前提（ログイン済み X API や Grok 連携を持っていない前提で動く、という一文）は、X API を持つ環境では自分の環境に合わせて読み替えてください。取得手順自体はそのまま使えます
- fxtwitter / fixupx は X社公式ではなく、第三者（FxEmbed）の埋め込みプロキシです

## ライセンス

MIT（予定 — 公開リポジトリ作成時に確定）

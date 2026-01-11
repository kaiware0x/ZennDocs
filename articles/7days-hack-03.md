---
title: "7日間でハッキングをはじめる本 - 勉強記録3"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

https://www.amazon.co.jp/7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC-TryHackMe%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%BA%AB%E4%BD%93%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8B%E6%94%BB%E6%92%83%E6%89%8B%E6%B3%95%E3%81%A8%E8%84%86%E5%BC%B1%E6%80%A7-%E9%87%8E%E6%BA%9D-%E3%81%AE%E3%81%BF%E3%81%9E%E3%81%86-ebook/dp/B0D8N3YMF9?crid=1LY91B54S5EQV&dib=eyJ2IjoiMSJ9.eZaruKe6nzn8EhYSSGQyXku5lGa50ttxoBZ0YHzpuxXKJ0kj7xDzjHJtlbgCUbWstqmYrBmDgkoJc9p0AZ3MYon9-Fx83NC2A4hmQ78U4RfxhDtn7AOTcaDXod1MZ3BAx-4Wl-BeVZjlWsZHHM1JWLBqddGL36oAxsKRhH1lhfWhcJNvwFbGugI4ocAmh2c2ryI50HA39iWcDaqlRBfaQ9SWfNUXlmJmMj1_h6Zb8LSHUXOJoYXcD27L2Tyj1iZtvnux6PdPHNYx8TfKJ0OAAVyj0uopYBUzjtBN5Bj53u0.3PoO-cuvREBAmVbZf9tXUMxpCiE6cNCoLg_-Hfz_MPg&dib_tag=se&keywords=7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC&qid=1767486791&sprefix=%2Caps%2C243&sr=8-1&linkCode=ll1&tag=kaiware0x-22&linkId=97eef18fec23fbcfc0d7886d32f5fca4&language=ja_JP&ref_=as_li_ss_tl

## 取り組んでいる課題

https://tryhackme.com/room/owaspjuiceshop

## OWASP

`OWASP (Open Worldwide Application Security Project, オワスプ) ` は、ソフトウェアのセキュリティを向上させる開発者・技術者・エヴァンジェリストのコミュニティ。
脆弱性をあえて残した、いわゆる ”やられサイト” がたくさん公開されている。

### OWASP Top 10

OWASPのプロジェクトの1つ。
Webアプリにおける重大なセキュリティリスクをまとめたもの。

## ローカルプロキシ
通常のHTTP通信は以下のような流れ。

```
ローカルマシン(自分のPC) のブラウザ
↕ HTTPメッセージ
ターゲットマシン(Webサーバ)
```

ローカルプロキシを咬ませると次のようになる。
ローカルプロキシが仲介してくれ、通信を書き換えたり再送したりしてくれる。

```
ローカルマシン(自分のPC) のブラウザ
↕ HTTPメッセージ
ローカルマシン(自分のPC) のローカルプロキシ
↕ HTTPメッセージ
ターゲットマシン(Webサーバ)
```

## Burp Suite

ローカルプロキシを利用するためのツール。

## SQLインジェクション

別名: SQL注入、英名: SQL Injection

ログインフォームなどにSQL構文を入力することで、様々な悪事を働くこと。

ログイン時のメールアドレス入力欄にSQLを入力し、パスワード認証をスキップして任意のユーザでログインすることが可能。
例えば、メールアドレス欄に以下のように入力する。

```
' or 1=1 --
```

この場合、データベースからユーザを取得するSQLに対し、

- 先頭の `'` でメールアドレス文字列を閉じ、
- `or 1=1` で常に条件文がTRUEになるようにし、
- `--` で、これ以降をコメントアウトする。

末尾のコメントアウトによって、パスワードが一致するかどうかの判定をスキップしている。

上記のインジェクションではデータベース上の一番上に登録されているユーザにログイン可能。
一番上は管理者の可能性が高いため、これでログインされると、あれやこれやWebサービスがめちゃくちゃにされてしまう。

また、以下のインジェクションではデータベースの上からN番目のユーザにログインすることが可能。

```
' or 1=1 ORDER BY id ASC LIMIT 1 OFFSET <任意の数字N>--
```

先程より色々書いてあるが、

- `ORDER BY id` でid順に並び替え
- `ASC` で昇順に並び替え (Ascending)
- `LIMIT 1` でデータベース1エントリのみ取得
- `OFFSET <任意の数字N>` でデータベースNエントリ分スキップ

ということになり、`N` に色々数字を入れて試して行けば、あらゆるユーザにログインすることが可能になる。(怖いね)

## リンク

前回：

https://zenn.dev/kaiware0x/articles/7days-hack-02

次回：

https://zenn.dev/kaiware0x/articles/7days-hack-04

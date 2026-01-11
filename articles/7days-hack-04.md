---
title: "7日間でハッキングをはじめる本 - 勉強記録4"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

https://www.amazon.co.jp/7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC-TryHackMe%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%BA%AB%E4%BD%93%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8B%E6%94%BB%E6%92%83%E6%89%8B%E6%B3%95%E3%81%A8%E8%84%86%E5%BC%B1%E6%80%A7-%E9%87%8E%E6%BA%9D-%E3%81%AE%E3%81%BF%E3%81%9E%E3%81%86-ebook/dp/B0D8N3YMF9?crid=1LY91B54S5EQV&dib=eyJ2IjoiMSJ9.eZaruKe6nzn8EhYSSGQyXku5lGa50ttxoBZ0YHzpuxXKJ0kj7xDzjHJtlbgCUbWstqmYrBmDgkoJc9p0AZ3MYon9-Fx83NC2A4hmQ78U4RfxhDtn7AOTcaDXod1MZ3BAx-4Wl-BeVZjlWsZHHM1JWLBqddGL36oAxsKRhH1lhfWhcJNvwFbGugI4ocAmh2c2ryI50HA39iWcDaqlRBfaQ9SWfNUXlmJmMj1_h6Zb8LSHUXOJoYXcD27L2Tyj1iZtvnux6PdPHNYx8TfKJ0OAAVyj0uopYBUzjtBN5Bj53u0.3PoO-cuvREBAmVbZf9tXUMxpCiE6cNCoLg_-Hfz_MPg&dib_tag=se&keywords=7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC&qid=1767486791&sprefix=%2Caps%2C243&sr=8-1&linkCode=ll1&tag=kaiware0x-22&linkId=97eef18fec23fbcfc0d7886d32f5fca4&language=ja_JP&ref_=as_li_ss_tl

## 取り組んでいる課題

https://tryhackme.com/room/owaspjuiceshop

## 環境立ち上げ

```
$ thm
$ burpsuite
```

課題リンクから仮想マシン起動し、IPアドレスを確認。
`burpsuite` からChromiumブラウザを起動し、コピーしたIPアドレスのサイトに移動する。

## OSINT

`Open Source Intelligence` の略。

SNSなどに公開されている情報など、合法的に入手できる公開情報を収集・分析し、意思決定につなげる活動のこと。

要するに、我々目線だと **”SNSにペットの名前や住んでいる地域などを書いていると、ハッキングの際にその情報を利用されてしまうかもよ”** ということ。

## Poison Null Byte (ヌルバイト攻撃)

一言で言うと「コンピュータが『文字列の終わり』を判断する方法のズレ」を突いた攻撃手法。

TryHackMeの課題を例にして、「`package.json.bak` とURLに入力してもダウンロード出来なくて、`package.json.bak%2500.md` と入力するとダウンロード出来る」という具体例について解説する。

下図は `package.json.bak` を直接ダウンロードしようとしたときの警告

![](/images/7days-hack-04/2026-01-11-11-41-11.png)


### 攻撃の流れ

- [クライアント] WebブラウザのURLに `http://<IP>/ftp/package.json.bak%2500.md` と入力してEnter。そのままサーバに送信される。
- [サーバ] PHPやNode.jsで書かれたサーバアプリが待ち受けている。このサーバアプリが `package.json.bak%2500.md` を `package.json.bak%00.md` にデコードする。(%25 は ”％” という文字そのものを表す)
- [サーバ] サーバアプリが、OSに対して「 `package.json.bak%00.md` というファイルを探して！」 とお願いする。
- [サーバ] OSは大体C言語で書かれており、 `%00` をヌル文字と判断するため、 `package.json.bak%00.md` は `package.json.bak[NULL].md` と解釈され、NULL以降が無視される。
- [サーバ] OS が `package.json.bak` というファイルをサーバアプリに返し、サーバアプリがクライアントに  `package.json.bak` を返す。

この流れで、本来ダウンロードできなかったはずの `package.json.bak` ファイルがダウンロードできてしまう。

## クロスサイトスクリプティング

英名： XSS, Cross-site scripting

Webアプリ上でJavaScriptコードを不正に実行できる脆弱性。

例えば、検索ボックスに

```html
<iframe src="javascript:alert(`xss`)">.)
```

と打ち込むと、Alert画面が出てきてしまう。

## リンク

前回：

https://zenn.dev/kaiware0x/articles/7days-hack-03

次回：

備えよう

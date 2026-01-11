---
title: "7日間でハッキングをはじめる本 - 勉強記録4"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

[![](/images/7days-hack-04/2026-01-11-12-02-12.png =250x)](https://amzn.to/4aP6SeL)
*7日間でハッキングをはじめる本 TryHackMeを使って身体で覚える攻撃手法と脆弱性*

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

---
title: "7日間でハッキングをはじめる本 - 勉強記録2"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

https://www.amazon.co.jp/7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC-TryHackMe%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%BA%AB%E4%BD%93%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8B%E6%94%BB%E6%92%83%E6%89%8B%E6%B3%95%E3%81%A8%E8%84%86%E5%BC%B1%E6%80%A7-%E9%87%8E%E6%BA%9D-%E3%81%AE%E3%81%BF%E3%81%9E%E3%81%86-ebook/dp/B0D8N3YMF9?crid=1LY91B54S5EQV&dib=eyJ2IjoiMSJ9.eZaruKe6nzn8EhYSSGQyXku5lGa50ttxoBZ0YHzpuxXKJ0kj7xDzjHJtlbgCUbWstqmYrBmDgkoJc9p0AZ3MYon9-Fx83NC2A4hmQ78U4RfxhDtn7AOTcaDXod1MZ3BAx-4Wl-BeVZjlWsZHHM1JWLBqddGL36oAxsKRhH1lhfWhcJNvwFbGugI4ocAmh2c2ryI50HA39iWcDaqlRBfaQ9SWfNUXlmJmMj1_h6Zb8LSHUXOJoYXcD27L2Tyj1iZtvnux6PdPHNYx8TfKJ0OAAVyj0uopYBUzjtBN5Bj53u0.3PoO-cuvREBAmVbZf9tXUMxpCiE6cNCoLg_-Hfz_MPg&dib_tag=se&keywords=7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC&qid=1767486791&sprefix=%2Caps%2C243&sr=8-1&linkCode=ll1&tag=kaiware0x-22&linkId=97eef18fec23fbcfc0d7886d32f5fca4&language=ja_JP&ref_=as_li_ss_tl

## 取り組んでいる課題

https://tryhackme.com/room/basicpentestingjt

https://tryhackme.com/room/blue

## nmapコマンドのオプション

nmapコマンドで、マシンで開いているポートを調査(ポートスキャン)できる。
コマンドのオプション：

- `-sV`
  - 開いているポートのサービスバージョンを自動的に特定してくれる。
  - これにより、特定のポートで動作しているサービスの詳細情報を取得できる。
- `-Pn`
  - ICPMリクエストの送信をスキップしてポートスキャンを実行する。
  - マシンによってはICPMリクエストをブロックしており、ポートスキャン実行が始まらない場合があるため。
- `-oN`
  - スキャン結果を任意のファイル名で保存する。
- `-v`
  - コマンドの処理の進捗が表示される。 (Verbose)
- `--script vuln`
  - nmapに付属するスクリプトを利用して、脆弱性の検出やブルートフォース攻撃を行うことができる


コマンド例：

```sh
$ nmap -sV -Pn -oN nmap.txt -v <ターゲットIPアドレス>  --script vuln
```

## ICPMリクエストとは

ICMPは、`Internet Control Message Protocol`の略。

主に、以下の2パターンで使用される。

1. 「生きてる？」と確認する時（エコーリクエスト）
   - ping コマンドで「おーい、聞こえる？」と呼びかけて、相手から返事（エコーリプライ）が来るかを確認する。
2. 「トラブルが起きた」と報告する時
   - データが目的地に届かなかったときに、ルーターなどの機器が「ごめん、道が塞がってて届けられなかったよ！」と教えてくれる。


## rockyou.txt が見つからない？

WSLの Kali Linux で作業している場合、 /usr/share/wordlists フォルダが存在しなかった。
少し調べたところ、WSL の Kali Linux は必要最低限のパッケージしか入っていないようだ。
以下のコマンドを実行して、フルサイズの Kali Linux をインストール出来る。

```sh
$ sudo apt install -y kali-linux-large
```

## GUI操作出来るようにする

参考：
https://zenn.dev/forwardone/articles/f30a0ee8ffd1c2

次のコマンドでデスクトップ環境のインストール→起動ができる。

```sh
$ sudo apt install -y kali-win-kex
$ kex --win -s -m
```

## hydra - 総当り攻撃ツール

hydra (ヒドラ) は、例えばSSH接続での認証パスワードを総当り攻撃できるようなツール。
以下のようなコマンドで、SSHプロトコルの攻撃が可能。

```sh
$ hydra -l <ユーザ名> -P <辞書ファイルパス> ssh://<IPアドレス> -t <並列数>
```

## RCE - Remote Code Execution

RCEとは、対象のコンピュータに外部から不正なコードを送り込み、実行できてしまうという脆弱性のこと。
一番怖いやつやん。

書籍では、

> この脆弱性をうまく悪用できればターゲットマシンを遠隔操作できるようになりますし、他の端末に横展開したり機密情報を探したりと夢が広がります。つまり、ハッカーにとっては非常においしいネタが見つかったといえます。

と書かれており、なんだか嬉しそう。

## エクスプロイト

攻撃に使うコードのこと。また実際に攻撃を行うこと。

## Metasploit Framework

エクスプロイトを行うためのツール。
Kali Linux では `msfconsole` コマンドが標準でインストールされている。

起動してみるとこんな感じ。派手なウェルカム画面が出てくる。

![](/images/7days-hack-02/2026-01-11-11-30-16.png)

## ハッシュからパスワードを解析する

書籍を進めていくと、ターゲットWindows端末の脆弱性を悪用し、管理者権限でログインが出来る。
ここで、脆弱性が塞がれてしまったときの抜け道を用意するため、端末内ユーザのパスワードを解析しておく。

**meterpreter** シェルを実行中に、 **hashdump** コマンドを実行すると、ユーザのパスワードのハッシュ値が得られる。
得られるのはハッシュ値のみ。(念押し)

ここからどうにかしてパスワードを知りたいが、一般的には
「パスワード→ハッシュ値」
を知ることは出来ても、
「ハッシュ値→パスワード」
の復元は不可能とされている。

しかし、**レインボーテーブル**を用いると不可能を可能にする。
仕組みとしては単純で、あらかじめ
**「パスワード→ハッシュ値」**
の組み合わせを大量に保持しておき、ハッシュ値から元のパスワードを逆引き出来るようにしておくだけ。

以下のサイトから、よく使われるレインボーテーブルを利用できる。

https://crackstation.net/

## ターゲットWindows端末にリモートデスクトップ接続する

ターゲットWindows端末は **3389 番ポート**が空いていた。
このポートは **RDP (Remote Desktop Protocol)** のためのポート。

以下の xfreerdp3 コマンドで、ターゲットWindows端末にリモートデスクトップ接続ができる。

```sh
$ xfreerdp3 /u:<ユーザ名> /v:<IPアドレス> /p:<パスワード> /cert:ignore /sec:rdp
```

![](/images/7days-hack-02/2026-01-11-11-32-03.png)


## リンク

前回：

https://zenn.dev/kaiware0x/articles/7days-hack-01

次回：

https://zenn.dev/kaiware0x/articles/7days-hack-03

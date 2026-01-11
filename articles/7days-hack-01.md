---
title: "7日間でハッキングをはじめる本 - 勉強記録1"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

https://www.amazon.co.jp/7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC-TryHackMe%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%BA%AB%E4%BD%93%E3%81%A7%E8%A6%9A%E3%81%88%E3%82%8B%E6%94%BB%E6%92%83%E6%89%8B%E6%B3%95%E3%81%A8%E8%84%86%E5%BC%B1%E6%80%A7-%E9%87%8E%E6%BA%9D-%E3%81%AE%E3%81%BF%E3%81%9E%E3%81%86-ebook/dp/B0D8N3YMF9?crid=1LY91B54S5EQV&dib=eyJ2IjoiMSJ9.eZaruKe6nzn8EhYSSGQyXku5lGa50ttxoBZ0YHzpuxXKJ0kj7xDzjHJtlbgCUbWstqmYrBmDgkoJc9p0AZ3MYon9-Fx83NC2A4hmQ78U4RfxhDtn7AOTcaDXod1MZ3BAx-4Wl-BeVZjlWsZHHM1JWLBqddGL36oAxsKRhH1lhfWhcJNvwFbGugI4ocAmh2c2ryI50HA39iWcDaqlRBfaQ9SWfNUXlmJmMj1_h6Zb8LSHUXOJoYXcD27L2Tyj1iZtvnux6PdPHNYx8TfKJ0OAAVyj0uopYBUzjtBN5Bj53u0.3PoO-cuvREBAmVbZf9tXUMxpCiE6cNCoLg_-Hfz_MPg&dib_tag=se&keywords=7%E6%97%A5%E9%96%93%E3%81%A7%E3%83%8F%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%82%92%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E6%9C%AC&qid=1767486791&sprefix=%2Caps%2C243&sr=8-1&linkCode=ll1&tag=kaiware0x-22&linkId=97eef18fec23fbcfc0d7886d32f5fca4&language=ja_JP&ref_=as_li_ss_tl

## 最小権限の原則

必要なユーザに必要な権限だけ与え、情報資産を守ること。

## パッケージバージョンアップ時にエラーが起きたとき

以下のコマンドで解決できる場合がある。(必ずでは無い)

```sh
$ sudo apt update --fix-missing
```

## TryHackMe内でのVPN設定ファイルの場所

書籍では、TryHackMeにログインしてから、自身のアイコンをクリックして「Access」から設定ファイルをダウンロード出来ると書かれている。
しかし、TryHackMeの仕様が変わったらしく、ダウンロードボタンの場所が変わっていた。

プロフィールアイコンをクリックして「Manage Account」を選択。

![](/images/7days-hack-01/2026-01-11-11-22-09.png)

下部の「Download Regular Cofiguration File」をクリックすればダウンロードできる。
VPNサーバの選択が出来ないが、利用中のマシンのリージョンから自動的に選択されるとのこと。

![](/images/7days-hack-01/2026-01-11-11-22-46.png)

## コマンドエイリアスの設定

VPN起動コマンドは長くて覚えづらいので、エイリアスを設定しておくと便利。

```sh
$ cd ~

# 既存のzsh設定をバックアップ
$ cp .zshrc .zshrc-bak

# エイリアスを .zshrc に追記する。 <ファイル名>は各々の環境で異なる。
$ echo "alias thm='sudo openvpn $HOME/vpn/<ファイル名>.ovpn'" >> .zshrc

# エイリアス設定が行われているか確認
$ alias
thm='sudo openvpn $HOME/vpn/<ファイル名>.ovpn'

# 以降は thm コマンドでVPN接続を行える。
$ thm
```


## リンク

前回：

なし

次回：

https://zenn.dev/kaiware0x/articles/7days-hack-02

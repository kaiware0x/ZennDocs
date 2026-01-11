---
title: "7日間でハッキングをはじめる本 - 勉強記録1"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

https://amzn.to/3LdmKNO

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

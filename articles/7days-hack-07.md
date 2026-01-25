---
title: "7日間でハッキングをはじめる本 - 勉強記録7"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["linux", "kalilinux", "wsl2", "security"]
published: true
---

こちらの本を勉強中に気になった用語や作業をまとめていきます。

[![](/images/7days-hack-07/2026-01-11-12-02-12.png =250x)](https://amzn.to/4aP6SeL)
**7日間でハッキングをはじめる本 TryHackMeを使って身体で覚える攻撃手法と脆弱性**

## 取り組んでいる課題

WordPressの脆弱性を学べる。

https://tryhackme.com/room/blog

## ステガノグラフィ

あるデータを他のデータの中に隠す技術。

## steghide

ステガノグラフィで隠されたデータを解析するツール。

```sh
$ sudo apt install steghide

# 指定したファイルから隠されたデータを抽出する。
$ steghide extract -sf <ファイルパス>
```

## WPScan

WordPress専用のセキュリティスキャナ。`wpscan`コマンドで利用できる。
Kali Linux には標準でインストールされている。

```sh
# URLを指定し、WordPressのユーザ情報を探索する
$ wpscan --url <URL> --enumerate u
```

特定のユーザに対し辞書攻撃を行うこともできる。

```sh
$ wpscan --url <URL> -U <ユーザ名> -P <辞書ファイルパス> --password-attack wp-login -t 4
```

## LinPEAS

ターゲットマシン上の設定ミスを教えてくれるシェルスクリプト。
ターゲットマシンへの初期侵入が終わった後、ターゲットマシン上で実行すると、権限昇格出来そうな設定ミスを教えてくれる。

## ltraceコマンド

プログラムを実行中、どんなライブラリ関数を呼び出しているかを追跡して表示してくれるコマンド。

試しに `ls` コマンドをトレースしてみた。
裏ではたくさんのライブラリ関数が呼び出されていることが分かる。

```sh
$ ltrace ls
strrchr("ls", '/')                                              = nil
setlocale(LC_ALL, "")                                           = "en_US.UTF-8"
bindtextdomain("coreutils", "/usr/share/locale")                = "/usr/share/locale"
textdomain("coreutils")                                         = "coreutils"
__cxa_atexit(0x5e8e248bae00, 0, 0x5e8e248d2008, 0)              = 0
getopt_long(1, 0x7ffd8f46a538, "abcdfghiklmnopqrstuvw:xABCDFGHI:"..., 0x5e8e248d1340, -1) = -1
getenv("LS_BLOCK_SIZE")                                         = nil
getenv("BLOCK_SIZE")                                            = nil
getenv("BLOCKSIZE")                                             = nil
getenv("POSIXLY_CORRECT")                                       = nil
getenv("BLOCK_SIZE")                                            = nil
isatty(1)                                                       = 0
getenv("QUOTING_STYLE")                                         = nil
__errno_location()                                              = 25
memcpy(0x5e8e46c10c10, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 56) = 0x5e8e46c10c10
__errno_location()                                              = 25
memcpy(0x5e8e46c10c50, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 56) = 0x5e8e46c10c50
getenv("TZ")                                                    = nil
strlen(".")                                                     = 1
memcpy(0x5e8e46c15ea0, ".\0", 2)                                = 0x5e8e46c15ea0
__errno_location()                                              = 25
opendir(".")                                                    = {3}
readdir({3})                                                    = {43229, "."}
readdir({3})                                                    = {33170, "rabbit_hole.txt"}
strlen("rabbit_hole.txt")                                       = 15
memcpy(0x5e8e46c1df00, "rabbit_hole.txt\0", 16)                 = 0x5e8e46c1df00
readdir({3})                                                    = {631867, "check-this.png"}
strlen("check-this.png")                                        = 14
memcpy(0x5e8e46c1df20, "check-this.png\0", 15)                  = 0x5e8e46c1df20
readdir({3})                                                    = {43335, "ltrace_ls_output.txt"}
strlen("ltrace_ls_output.txt")                                  = 20
memcpy(0x5e8e46c1df40, "ltrace_ls_output.txt\0", 21)            = 0x5e8e46c1df40
readdir({3})                                                    = {43560, "linpeas_output.txt"}
strlen("linpeas_output.txt")                                    = 18
memcpy(0x5e8e46c1df60, "linpeas_output.txt\0", 19)              = 0x5e8e46c1df60
readdir({3})                                                    = {43350, "linpeas.sh"}
strlen("linpeas.sh")                                            = 10
memcpy(0x5e8e46c1df80, "linpeas.sh\0", 11)                      = 0x5e8e46c1df80
readdir({3})                                                    = {44392, "nmap.txt"}
strlen("nmap.txt")                                              = 8
memcpy(0x5e8e46c1dfa0, "nmap.txt\0", 9)                         = 0x5e8e46c1dfa0
readdir({3})                                                    = {631843, "tswift.mp4"}
strlen("tswift.mp4")                                            = 10
memcpy(0x5e8e46c1dfc0, "tswift.mp4\0", 11)                      = 0x5e8e46c1dfc0
readdir({3})                                                    = {40457, ".."}
readdir({3})                                                    = {47129, "Alice-White-Rabbit.jpg"}
strlen("Alice-White-Rabbit.jpg")                                = 22
memcpy(0x5e8e46c1dfe0, "Alice-White-Rabbit.jpg\0", 23)          = 0x5e8e46c1dfe0
readdir({3})                                                    = {43254, "Billy_Joel_Termination_May20-202"...}
strlen("Billy_Joel_Termination_May20-202"...)                   = 37
memcpy(0x5e8e46c1e000, "Billy_Joel_Termination_May20-202"..., 38) = 0x5e8e46c1e000
readdir({3})                                                    = nil
closedir({3})                                                   = 0
realloc(nil, 216)                                               = 0x5e8e46c15ec0
_setjmp(0x5e8e248d2320, 1, 0xffffffff, 0x5e8e46c15f08)          = 0
__errno_location()                                              = 0
strcoll("Alice-White-Rabbit.jpg", "Billy_Joel_Termination_May20-202"...) = -13
__errno_location()                                              = 0
strcoll("tswift.mp4", "Alice-White-Rabbit.jpg")                 = 116
__errno_location()                                              = 0
strcoll("tswift.mp4", "Billy_Joel_Termination_May20-202"...)    = 103
memcpy(0x5e8e46c15f00, "\0\022\301F\216^\0\0", 8)               = 0x5e8e46c15f00
__errno_location()                                              = 0
strcoll("linpeas.sh", "nmap.txt")                               = -30
memcpy(0x5e8e46c15f10, "0\021\301F\216^\0\0", 8)                = 0x5e8e46c15f10
__errno_location()                                              = 0
strcoll("linpeas.sh", "Alice-White-Rabbit.jpg")                 = 114
__errno_location()                                              = 0
strcoll("linpeas.sh", "Billy_Joel_Termination_May20-202"...)    = 101
__errno_location()                                              = 0
strcoll("linpeas.sh", "tswift.mp4")                             = -2
__errno_location()                                              = 0
strcoll("nmap.txt", "tswift.mp4")                               = -2
__errno_location()                                              = 0
strcoll("ltrace_ls_output.txt", "linpeas_output.txt")           = 3
__errno_location()                                              = 0
strcoll("rabbit_hole.txt", "check-this.png")                    = 91
__errno_location()                                              = 0
strcoll("check-this.png", "linpeas_output.txt")                 = -90
__errno_location()                                              = 0
strcoll("rabbit_hole.txt", "linpeas_output.txt")                = 1
__errno_location()                                              = 0
strcoll("rabbit_hole.txt", "ltrace_ls_output.txt")              = 1
memcpy(0x5e8e46c15f20, " \r\301F\216^\0\0", 8)                  = 0x5e8e46c15f20
__errno_location()                                              = 0
strcoll("check-this.png", "Alice-White-Rabbit.jpg")             = 24
__errno_location()                                              = 0
strcoll("check-this.png", "Billy_Joel_Termination_May20-202"...) = 11
__errno_location()                                              = 0
strcoll("check-this.png", "linpeas.sh")                         = -90
__errno_location()                                              = 0
strcoll("linpeas_output.txt", "linpeas.sh")                     = -2
__errno_location()                                              = 0
strcoll("ltrace_ls_output.txt", "linpeas.sh")                   = 3
__errno_location()                                              = 0
strcoll("ltrace_ls_output.txt", "nmap.txt")                     = -30
__errno_location()                                              = 0
strcoll("rabbit_hole.txt", "nmap.txt")                          = 1
__errno_location()                                              = 0
strcoll("rabbit_hole.txt", "tswift.mp4")                        = -1
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("Alice-White-Rabbit.jpg")                                = 22
fwrite_unlocked("Alice-White-Rabbit.jpg", 1, 22, 0x79354fe935c0) = 22
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("Billy_Joel_Termination_May20-202"...)                   = 37
fwrite_unlocked("Billy_Joel_Termination_May20-202"..., 1, 37, 0x79354fe935c0) = 37
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("check-this.png")                                        = 14
fwrite_unlocked("check-this.png", 1, 14, 0x79354fe935c0)        = 14
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("linpeas_output.txt")                                    = 18
fwrite_unlocked("linpeas_output.txt", 1, 18, 0x79354fe935c0)    = 18
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("linpeas.sh")                                            = 10
fwrite_unlocked("linpeas.sh", 1, 10, 0x79354fe935c0)            = 10
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("ltrace_ls_output.txt")                                  = 20
fwrite_unlocked("ltrace_ls_output.txt", 1, 20, 0x79354fe935c0)  = 20
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("nmap.txt")                                              = 8
fwrite_unlocked("nmap.txt", 1, 8, 0x79354fe935c0)               = 8
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("rabbit_hole.txt")                                       = 15
fwrite_unlocked("rabbit_hole.txt", 1, 15, 0x79354fe935c0)       = 15
__errno_location()                                              = 0
__ctype_get_mb_cur_max()                                        = 6
strlen("tswift.mp4")                                            = 10
fwrite_unlocked("tswift.mp4", 1, 10, 0x79354fe935c0)            = 10
__fpending(0x79354fe935c0)                                      = 163
fileno(0x79354fe935c0)                                          = 1
__freading(0x79354fe935c0, 0, 1, 0)                             = 0
__freading(0x79354fe935c0, 0, 0, 0)                             = 0
fflush(0x79354fe935c0)                                          = 0
fclose(0x79354fe935c0)                                          = 0
__fpending(0x79354fe934e0)                                      = 0
fileno(0x79354fe934e0)                                          = 2
__freading(0x79354fe934e0, 0, 0x79354fe90e00, 0x6f51)           = 0
__freading(0x79354fe934e0, 0, 0, 0x6f51)                        = 0
fflush(0x79354fe934e0)                                          = 0
fclose(0x79354fe934e0)                                          = 0
+++ exited (status 0) +++
```


## リンク

前回：

https://zenn.dev/kaiware0x/articles/7days-hack-06

次回：

完
<!-- https://zenn.dev/kaiware0x/articles/7days-hack-08 -->

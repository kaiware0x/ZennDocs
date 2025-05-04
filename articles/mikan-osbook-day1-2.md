---
title: "ゼロからのOS自作入門 メモ (Day1-2)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: false
---

みかん本Day1その2。

[前回の記事]()が長くなってしまったので続きを書いていく。

## 備考：ループバックデバイス

`mount`コマンドの`-o loop`は**ループバックデバイス**を指定するオプション。
ループバックデバイスとはLinuxの仮想デバイスの一種。
普通のファイル（例：disk.img）をブロックデバイス（ハードディスクのような扱い）としてマウントするための仕組み。

概念図：

![](mikan-osbook-day1-2/loopback-device.drawio.svg)

## 備考：文字コード

## 備考：多バイト数値の並べ方

- リトルエンディアン
- ビッグエンディアン

## OS起動までの流れ

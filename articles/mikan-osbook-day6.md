---
title: "ゼロからのOS自作入門 メモ (Day6)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: false
---

こんにちは、**kaiware**です。

みかん本Day6(第6章)について、各種用語や詰まった箇所のメモを残す。

前回はこちら。
https://zenn.dev/kaiware0x/articles/mikan-osbook-day5

# USBホストコントローラ

- USB機器とソフトウェア(OS)を橋渡しするための制御チップ
- マザーボードに搭載されている

# ドライバ Driver

- ハードウェアを扱うためのソフトウェアのこと
- OSの一部
- USB関連のドライバ：
  - ターゲットドライバ
    - 個々のUSB機器(USBターゲット)に搭載するドライバ
  - **ホストドライバ**(本書で扱う)
    - 中心となるコンピュータ(USBホスト)に搭載するドライバ

# 次回

備えよう

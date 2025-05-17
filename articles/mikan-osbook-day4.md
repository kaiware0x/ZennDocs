---
title: "ゼロからのOS自作入門 メモ (Day4)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: true
---

こんにちは、**kaiware**です。

みかん本Day4(第4章)について、各種用語や詰まった箇所のメモを残す。

前回はこちら。
https://zenn.dev/kaiware0x/articles/mikan-osbook-day3


# CプログラムからC++関数を呼び出す

参照を受け取るC++関数をCプログラムから呼び出す際には、C側ではポインタ渡しをすれば良い。
これはコンパイラの仕様で決まっている。
今回の場合、こちらの仕様に準拠している：`System V AMD64 ABI`

# 配置new

- 通常のnewはインスタンスをヒープ領域に確保する
  - プログラマがメモリ領域の指定はできず、OSがメモリを割り当ててくれる
  - ヒープ領域：関数の実行が終了しても破棄されない領域
  - newを使わないで生成したインスタンスはスタック領域に配置される
- **配置newはプログラマがメモリ領域を指定できる**
  - 現状のMikanOSはメモリ管理機能を実装していないので、プログラマが指定する必要がある

```cpp
void* operator new(size_t size, void* buf) {
    return buf;
}

void operator delete(void* obj) noexcept {
}
```

# newとmalloc()の違い

- new: コンストラクタが呼ばれる
    - 初期化される
- malloc(): コンストラクタが呼ばれない
    - 初期化されず、ゴミデータが入る

# vtable

- 仮想関数テーブル, Virtual function TABLE
- 仮想関数のポインタ表
- 各クラスごとに存在する

# カーネル読み込み処理の修正

day04c時点の`MikanLoaderPkg/Main.c`にはバグがあり、カーネル用のメモリの大きさを計算する処理が間違っている。
`kernel.elf`の**プログラムヘッダ**を見れば、必要なメモリサイズがわかる。

```log
kaiware@tslap:~/workspace/mikanos/kernel$ readelf -l kernel.elf

Elf file type is EXEC (Executable file)
Entry point 0x1011d0
There are 5 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000100040 0x0000000000100040
                 0x0000000000000118 0x0000000000000118  R      0x8
  LOAD           0x0000000000000000 0x0000000000100000 0x0000000000100000
                 0x00000000000001a8 0x00000000000001a8  R      0x1000
  LOAD           0x00000000000001b0 0x00000000001011b0 0x00000000001011b0
                 0x00000000000001c9 0x00000000000001c9  R E    0x1000
  LOAD           0x0000000000000380 0x0000000000102380 0x0000000000102380
                 0x0000000000000000 0x0000000000000018  RW     0x1000
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x0

 Section to Segment mapping:
  Segment Sections...
   00
   01     .rodata
   02     .text
   03     .bss
   04
readelf: Warning: Unrecognized form: 0x23
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x23
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
readelf: Warning: Unrecognized form: 0x22
kaiware@tslap:~/workspace/mikanos/kernel$
```

LOADセグメント情報まとめ：

| オフセット | 仮想Addr | ファイルサイズ | メモリサイズ | フラグ |
| ---------- | -------- | -------------- | ------------ | ------ |
| 0x000      | 0x100000 | 0x1a8          | 0x1a8        | R      |
| 0x1b0      | 0x1011b0 | 0x1c9          | 0x1c9        | R E    |
| 0x380      | 0x102380 | 0x000          | 0x018        | RW     |


- それぞれの LOADセグメント が 各セクションに対応している
    - 1つ目のLOAD → `.rodata`
    - 2つ目のLOAD → `.text`
    - 3つ目のLOAD → `.bss`など
        - bss: グローバル変数が配置されるセクション。
        - ここだけ `FileSiz` と `MemSiz` が一致していないが、今回`main.cpp`で定義したグローバル変数には初期値が無いため、 `FileSiz` はゼロで良い。
- 仮想Addrが`0x100000`から始まるのは`ld.lld`コマンドのオプションで`--image-base 0x100000`を指定したから。
- 仮想Addrが`0x1000`ずつ増えていくのは、ページサイズが`0x1000`(4KiB)だから。
    - ページアラインメントを行っている。
- フラグの意味：
    - R: Readable
    - W: Writable
    - E: Executable


本来読み込みたいメモリアドレス配分：

![](/images/mikan-osbook-day4/load-segment.drawio.png)

`Loader.efi`での処理：

![](/images/mikan-osbook-day4/process-load-segment.drawio.png)


実行結果(QEMUにて)：

![実行結果](/images/mikan-osbook-day4/image.png)


# 次回

https://zenn.dev/kaiware0x/articles/mikan-osbook-day5

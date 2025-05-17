---
title: "ゼロからのOS自作入門 メモ (Day5)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: true
---

こんにちは、**kaiware**です。

みかん本Day5(第5章)について、各種用語や詰まった箇所のメモを残す。

前回はこちら。
https://zenn.dev/kaiware0x/articles/mikan-osbook-day4


# フォントファイルの埋め込み

![](/images/mikan-osbook-day5/embed-font.drawio.png)

## GetFont()

```cpp
// #@@range_begin(hankaku_bin)
extern const uint8_t _binary_hankaku_bin_start;
extern const uint8_t _binary_hankaku_bin_end;
extern const uint8_t _binary_hankaku_bin_size;

const uint8_t* GetFont(char c) {
  auto index = 16 * static_cast<unsigned int>(c);
  if (index >= reinterpret_cast<uintptr_t>(&_binary_hankaku_bin_size)) {
    return nullptr;
  }
  return &_binary_hankaku_bin_start + index;
}
// #@@range_end(hankaku_bin)
```

- 埋め込んだ`hankaku.bin`ファイルは、ソースコード中では`_binary_hankaku_bin_*`という特別な名前でアクセスできる。
- `hankaku.bin`には、1bitごとに塗る・塗らない(文字の前景・背景)情報が含まれている。
  - 前景なら1, 背景なら0
- 引数の`char c`は1Byteで文字のIndexを表している
  - 16倍することで`hankaku.bin`中の実際のStart Indexがわかる
    - 1文字の縦サイズが16pxなので、16px間隔でStart Indexがある

hankaku.txt例：

```
0x41 'A'
........
...@....
...@....
..@.@...
..@.@...
..@.@...
.@...@..
.@...@..
.@...@..
.@@@@@..
@.....@.
@.....@.
@.....@.
@.....@.
........
........
```

## WriteAscii()

```cpp
// #@@range_begin(write_ascii)
void WriteAscii(PixelWriter& writer, int x, int y, char c, const PixelColor& color) {
  const uint8_t* font = GetFont(c);
  if (font == nullptr) {
    return;
  }
  for (int dy = 0; dy < 16; ++dy) {
    for (int dx = 0; dx < 8; ++dx) {
      if ((font[dy] << dx) & 0x80u) {
        writer.Write(x + dx, y + dy, color);
      }
    }
  }
}
// #@@range_end(write_ascii)
```

- 画面に表示するときには画像座標系になるため、左上から塗るpxを走査していく。
  - 画像座標系：左上が原点で、y軸プラスが下向き
- 各文字の1行が1Byte(8bit)、縦幅が2Byte(16bit)



各行のビット演算の解説：

![](/images/mikan-osbook-day5/write-ascii.drawio.png)

ここまでの実行結果(day05d):

![day05d 実行結果](/images/mikan-osbook-day5/image.png)

# コンソール

`Console`クラスは25行分の文字列しか保持しないようになっているため、`line 3~26`のみ表示される。
(末尾に改行が入っているため、25行目は空白になっている)

![day05e 実行結果](/images/mikan-osbook-day5/image-1.png)

# QEMUで実行するまで まとめ

```sh
cd ~

# C/C++環境のセットアップ
source ~/osbook/devenv/buildenv.sh

# カーネルのビルド
cd ~/workspace/mikanos/kernel
make
# ~/workspace/mikanos/kernel/kernel.elf ができていることを確認する

# edk2環境のセットアップ
cd ~/edk2
source edksetup.sh

# ブートローダーのビルド
build
# ~/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi ができていることを確認する

# QEMUで実行
~/osbook/devenv/run_qemu.sh ./Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi ~/workspace/mikanos/kernel/kernel.elf
```

# 次回

備えよう

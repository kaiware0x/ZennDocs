---
title: "ゼロからのOS自作入門 メモ (Day1-1)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: true
---

こんにちは。 **kaiware** です。

LLMが世を席巻している昨今、低レイヤー技術へ自分のコア技術をシフトしていくことで ~~競争から逃げ、~~ より付加価値の高い人材になれると思い自作OSにチャレンジする。
ここでは、書籍「ゼロからのOS自作入門」をなぞっていき、メモを残していく。
メインOSはWindows11を使い、その上にWSL2でUbuntu環境を構築して学習していく。

https://zero.osdev.jp

:::message
以下 **みかん本** と呼ぶ。
:::

## 動作環境

- Windows バージョン: 10.0.26100.3915
- WSL バージョン: 2.4.13.0
- カーネル バージョン: 5.15.167.4-1
- WSLg バージョン: 1.0.65
- MSRDC バージョン: 1.2.5716
- Direct3D バージョン: 1.611.1-81528511
- DXCore バージョン: 10.0.26100.1-240331-1435.ge-release
- Ubuntu 22.04 LTS

図にすると(多分)こんな感じ。QEMUというエミュレータ上で、今回開発する **MikanOS** を動かす。

![](/images/mikan-osbook-day1/DevenvOverview.drawio.png)

## Windowsでの準備

このリポジトリをClone。 **osbook**というフォルダ名にしておく。

https://github.com/uchan-nos/mikanos-build

ここを参考にWSLg環境を作成。

https://thinkit.co.jp/article/37792

:::message
みかん本にはWindows側にXサーバをインストールする必要があると書いてあったが、今はWSLgがあるのでインストールしなくて良い。
:::

Windowsにバイナリエディタ **Binary Editor Bz** をインストールする。

https://forest.watch.impress.co.jp/library/software/binaryeditbz/

## EFIファイルの作成

Bzでみかん本に載っているバイナリをちまちま打ち込む。

保存先： `C:\Work\data\BOOTX64.EFI`

また、Clone済みosbookフォルダをWSL/Ubuntu内の`$HOME`直下へコピーしておく。

## QEMUのインストール、Imageの作成

Ubuntuで以下コマンドを実行していく。

```sh
# QEMUのインストール
sudo apt install qemu-system

# homeディレクトリへEFIファイルをコピー
cd /mnt/c/Work/data
cp BOOTX64.EFI ~
cd ~
mkdir -p mnt # -pで親ディレクトリも作成できる

# disk imageの作成
qemu-img create -f raw disk.img 200M
# -> Formatting 'disk.img', fmt=raw size=209715200

# disk imageのフォーマット
mkfs.fat -n 'MIKAN OS' -s 2 -f 2 -R 32 -F 32 disk.img
# -> mkfs.fat 4.2 (2021-01-31)

# disk imageのMount
# mntフォルダの実体がdisk.imgファイルになる
sudo mount -o loop disk.img mnt

# disk.imgにEFIファイルを書き込み
sudo mkdir -p mnt/EFI/BOOT
sudo cp BOOTX64.EFI mnt/EFI/BOOT/BOOTX64.EFI
sudo umount mnt

```

## QEMU起動

```sh
# QEMUでEFIファイル起動
qemu-system-x86_64 \
    -drive if=pflash,file=$HOME/osbook/devenv/OVMF_CODE.fd \
    -drive if=pflash,file=$HOME/osbook/devenv/OVMF_VARS.fd \
    -hda disk.img
```

実行結果：

ちゃんと Hello World が表示されている。

![alt text](/images/mikan-osbook-day1/image.png)

ひとまず起動ができたので、続きは後日別記事へ。

**追記：続き書きました**

https://zenn.dev/kaiware0x/articles/mikan-osbook-day1-2

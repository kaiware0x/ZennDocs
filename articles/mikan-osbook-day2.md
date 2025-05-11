---
title: "ゼロからのOS自作入門 メモ (Day2)"
emoji: "🍊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os", "linux", "windows", "wsl"]
published: true
---

こんにちは、**kaiware**です。

みかん本Day2(第2章)について、各種用語や詰まった箇所のメモを残す。

前回はこちら。
https://zenn.dev/kaiware0x/articles/mikan-osbook-day1-2

# EDK II (EDK2)

- UEFIとその周辺のプログラムの開発キット
- もともとIntel主導で開発されていた
- OSS
    - https://github.com/tianocore/edk2
- MdePkg
    - 以下のヘッダを提供する
        - `#include <Uefi.h>`
        - `#include <Library/Uefi.h>`
- 他にも様々なPkg(ライブラリ)が格納されている


# EDK2を用いたUEFIアプリケーションの作成

大まかな手順：

1. 自前パッケージ(`MikanLoaderPkg`)の編集
   - ビルドに使うソースファイルを作成、指定
   - エントリーポイントになる関数を指定
   - EDK2の中の使用するライブラリを指定
2. edksetup.shの実行(初回のみ)
   - edkビルド設定ファイルを初期化する
4. edk2/Conf/target.txtの編集
   - ターゲットアーキテクチャなどの設定を行う
5. ビルド実行
   - EFIファイルが生成される
6. QEMUなどで動作確認


メモ：

- UEFIでPrintするときはワイド文字を使う必要がある
    - `Print(L"Hello, Mikan World!\n");`

## ブートローダーのビルド

```sh
cd $HOME/edk2
ln -s $HOME/workspace/mikanos/MikanLoaderPkg ./ # シンボリックリンクを張る
source edksetup.sh # ターミナル起動時のみ。環境変数がセットされる
build
# Output: $HOME/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi
```

## buildコマンドでのエラー

以下のエラーが出た。

```log
kaiware@tslap:~/edk2$ build
Build environment: Linux-5.15.167.4-microsoft-standard-WSL2-x86_64-with-glibc2.35
Build start time: 17:42:21, May.10 2025

WORKSPACE        = /home/kaiware/edk2
EDK_TOOLS_PATH   = /home/kaiware/edk2/BaseTools
CONF_PATH        = /home/kaiware/edk2/Conf
PYTHON_COMMAND   = /usr/bin/python3


Processing meta-data .
Architecture(s)  = X64
Build target     = DEBUG
Toolchain        = CLANG38

Active Platform          = /home/kaiware/edk2/MikanLoaderPkg/MikanLoaderPkg.dsc


build.py...
/home/kaiware/edk2/MikanLoaderPkg/MikanLoaderPkg.dsc(...): error 4000: Instance of library class [RegisterFilterLib] is not found
        in [/home/kaiware/edk2/MdePkg/Library/BaseLib/BaseLib.inf] [X64]
        consumed by module [/home/kaiware/edk2/MikanLoaderPkg/Loader.inf]


- Failed -
Build end time: 17:42:21, May.10 2025
Build total time: 00:00:00
```

こちらの2の方法を参考に解決した。

https://github.com/uchan-nos/os-from-zero/blob/main/faq.md#registerfilterlib-%E9%96%A2%E4%BF%82%E3%81%AE%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%A7-mikanloaderpkg-%E3%81%8C%E3%83%93%E3%83%AB%E3%83%89%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%9B%E3%82%93

具体的には、`MikanLoaderPkg/MikanLoaderPkg.dsc`に以下の行を追加した。

```diff
   PrintLib|MdePkg/Library/BasePrintLib/BasePrintLib.inf
+  RegisterFilterLib|MdePkg/Library/RegisterFilterLibNull/RegisterFilterLibNull.inf
   UefiBootServicesTableLib|MdePkg/Library/UefiBootServicesTableLib/UefiBootServicesTableLib.inf
```

再度buildコマンドを実行したところ、エラーなく終了し、以下のフォルダに`Loader.efi`ファイルが生成された。

```
$HOME/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efi
```

また次のコマンドで、ビルドした`Loader.efi`をQEMUで実行してみる。

```
cd $HOME/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64
$HOME/osbook/devenv/run_qemu.sh Loader.efi
```

`Hello, Mikan World!`と表示されているため、正しく動作している。

![Loader.efi 動作結果](/images/mikan-osbook-day2/image.png)

## メモリマップの取得

メモリマップを取得してファイル保存(csv)するように修正 (`git checkout osbook_day02b`)。

上記と同様に`Loader.efi`をビルドしてQEMUで実行。

実行結果：

![alt text](/images/mikan-osbook-day2/image-1.png)

```
map->buffer = 3FEA8A00, map->map_size = 000008A0
```

用意されている実行スクリプト`$HOME/osbook/devenv/run_qemu.sh`では、`disk.img`というイメージファイルが生成されるようになっている。
このイメージを`$HOME/mnt`にマウントし、出力されたメモリマップcsv(`memmap`)を表示してみる。

- `PhysicalStart`
    - メモリ領域の先頭アドレス(16進数)
- `NumberOfPages`
    - その領域のページ数(16進数)
    - UEFIにおいて、1Page = 4KiB
        - KiB = キビバイト, 1024Byte
- `Attribute`
    - そのメモリ領域が使える用途を示すビット集合

先程の`map->buffer = 3FEA8A00`と表を照らし合わせると、 **Index=39~40** の間にメモリマップ(16KiB分)が格納されていることがわかる。

| Index | Type | Type(name)             | PhysicalStart | NumberOfPages | Attribute |
| ----- | ---- | ---------------------- | ------------- | ------------- | --------- |
| 0     | 3    | EfiBootServicesCode    | 00000000      | 1             | F         |
| 1     | 7    | EfiConventionalMemory  | 00001000      | 9F            | F         |
| 2     | 7    | EfiConventionalMemory  | 00100000      | 700           | F         |
| 3     | A    | EfiACPIMemoryNVS       | 00800000      | 8             | F         |
| 4     | 7    | EfiConventionalMemory  | 00808000      | 8             | F         |
| 5     | A    | EfiACPIMemoryNVS       | 00810000      | F0            | F         |
| 6     | 4    | EfiBootServicesData    | 00900000      | B00           | F         |
| 7     | 7    | EfiConventionalMemory  | 01400000      | 3AB36         | F         |
| 8     | 4    | EfiBootServicesData    | 3BF36000      | 20            | F         |
| 9     | 7    | EfiConventionalMemory  | 3BF56000      | 2726          | F         |
| 10    | 1    | EfiLoaderCode          | 3E67C000      | 2             | F         |
| 11    | 4    | EfiBootServicesData    | 3E67E000      | A             | F         |
| 12    | 9    | EfiACPIReclaimMemory   | 3E688000      | 1             | F         |
| 13    | 4    | EfiBootServicesData    | 3E689000      | 1F7           | F         |
| 14    | 3    | EfiBootServicesCode    | 3E880000      | B4            | F         |
| 15    | A    | EfiACPIMemoryNVS       | 3E934000      | 12            | F         |
| 16    | 0    | EfiReservedMemoryType  | 3E946000      | 1C            | F         |
| 17    | 3    | EfiBootServicesCode    | 3E962000      | 10A           | F         |
| 18    | 6    | EfiRuntimeServicesData | 3EA6C000      | 5             | F         |
| 19    | 5    | EfiRuntimeServicesCode | 3EA71000      | 5             | F         |
| 20    | 6    | EfiRuntimeServicesData | 3EA76000      | 5             | F         |
| 21    | 5    | EfiRuntimeServicesCode | 3EA7B000      | 5             | F         |
| 22    | 6    | EfiRuntimeServicesData | 3EA80000      | 5             | F         |
| 23    | 5    | EfiRuntimeServicesCode | 3EA85000      | 7             | F         |
| 24    | 6    | EfiRuntimeServicesData | 3EA8C000      | 8F            | F         |
| 25    | 4    | EfiBootServicesData    | 3EB1B000      | 6F7           | F         |
| 26    | 7    | EfiConventionalMemory  | 3F212000      | 4             | F         |
| 27    | 4    | EfiBootServicesData    | 3F216000      | 6             | F         |
| 28    | 7    | EfiConventionalMemory  | 3F21C000      | 1             | F         |
| 29    | 4    | EfiBootServicesData    | 3F21D000      | 7FE           | F         |
| 30    | 7    | EfiConventionalMemory  | 3FA1B000      | 1             | F         |
| 31    | 3    | EfiBootServicesCode    | 3FA1C000      | 17F           | F         |
| 32    | 5    | EfiRuntimeServicesCode | 3FB9B000      | 30            | F         |
| 33    | 6    | EfiRuntimeServicesData | 3FBCB000      | 24            | F         |
| 34    | 0    | EfiReservedMemoryType  | 3FBEF000      | 4             | F         |
| 35    | 9    | EfiACPIReclaimMemory   | 3FBF3000      | 8             | F         |
| 36    | A    | EfiACPIMemoryNVS       | 3FBFB000      | 4             | F         |
| 37    | 4    | EfiBootServicesData    | 3FBFF000      | 201           | F         |
| 38    | 7    | EfiConventionalMemory  | 3FE00000      | 8D            | F         |
| 39    | 4    | EfiBootServicesData    | 3FE8D000      | 20            | F         |
| 40    | 3    | EfiBootServicesCode    | 3FEAD000      | 20            | F         |
| 41    | 4    | EfiBootServicesData    | 3FECD000      | 9             | F         |
| 42    | 3    | EfiBootServicesCode    | 3FED6000      | 1E            | F         |
| 43    | 6    | EfiRuntimeServicesData | 3FEF4000      | 84            | F         |
| 44    | A    | EfiACPIMemoryNVS       | 3FF78000      | 88            | F         |
| 45    | 6    | EfiRuntimeServicesData | FFC00000      | 400           | 1         |


## メモリマップとは？

メモリマップ（memory map）は、コンピュータの**メモリ空間がどのように使われているかを示す地図**のようなもの。

コンピュータのCPUは、**アドレス（番地）を使ってメモリにアクセス**するが、メモリ全体が自由に使えるわけではない。

### 🗺️ メモリは「用途ごとに区切られている」

たとえば、以下のように分かれて使われている（例：32bit OSの場合の典型的な配置）：

| 領域名                 | 役割                             |
| ---------------------- | -------------------------------- |
| 0x00000000〜0x000FFFFF | BIOSや割り込みテーブルなど       |
| 0x00100000〜0xBFFFFFFF | 実際のRAM（プログラムやデータ）  |
| 0xC0000000〜0xFFFFFFFF | OSカーネル・デバイスとの通信など |

### 🏠 メモリを「一軒家の間取り」で例えると…

* **玄関（低い番地）**：OSの初期設定や割り込み処理の番地（BIOSの領域）
* **リビング**：プログラムが実際に動くRAM
* **キッチン**：ヒープ領域（動的に使われるメモリ）
* **屋根裏部屋**：スタック（関数の呼び出し履歴）
* **屋根の上**：OSカーネルや周辺機器が住んでいる領域（ユーザーは勝手に触れない）

### 🧩 実際の用途の分類（簡略化）

| 領域名                | 用途                       | 備考               |
| --------------------- | -------------------------- | ------------------ |
| テキスト領域（.text） | プログラムの命令           | 読み取り専用       |
| データ領域（.data）   | 初期化済み変数             | 読み書き可能       |
| BSS領域（.bss）       | 初期化されていない変数     | 起動時に0で埋まる  |
| ヒープ（heap）        | `malloc()`などで動的に確保 | 拡張される領域     |
| スタック（stack）     | 関数呼び出しや一時変数     | 上から下へ積まれる |

### 🧪 なぜ重要？

* OS開発や組み込み開発では、**どのアドレスに何があるか**を把握する必要がある
* **メモリ保護**（セキュリティ）のため、領域を分けて管理する
* **デバイス制御**では、メモリ空間にマッピングされた**ハードウェアレジスタ**を操作する（メモリマップドI/O）

### 💡 具体例：`memmap` コマンド（UEFIブート時など）

自作OSやブートローダでは、UEFIから以下のようなメモリマップ情報を取得できる：

```
Address: 0x0000000000000000 - 0x0000000000009FFF : Available (RAM)
Address: 0x000000000000A000 - 0x000000000000FFFF : Reserved (BIOS)
Address: 0x0000000000010000 - ...               : Available (RAM)
```

これにより「使っていいメモリ」「使ってはいけないメモリ」が分かる。

### ✅ まとめ

| ポイント           | 説明                                                 |
| ------------------ | ---------------------------------------------------- |
| メモリマップとは？ | メモリ空間の用途や意味を示す一覧表                   |
| なぜ必要？         | OSやハードウェアが正しくメモリを使い分けるため       |
| 主な領域           | テキスト、データ、ヒープ、スタック、デバイス領域など |
| どこで使う？       | OS設計、デバイス制御、UEFI/BIOS開発、自作OSなど      |

## メモリディスクリプタとは？

メモリディスクリプタ（Memory Descriptor）は、**コンピュータの各メモリ領域(メモリマップ)の「住所・広さ・用途」を記録した一覧表の1行（1項目）** のこと。
特に **UEFI** や **OSのブート時** に使われる。

### ✅ そのメモリマップの「1項目」を指す！

**メモリの1領域の情報を1個分だけ表したデータ構造**が「メモリディスクリプタ」。

📦 イメージで言うと…

| 開始アドレス | サイズ | タイプ                             |
| ------------ | ------ | ---------------------------------- |
| `0x00000000` | 640KB  | `EfiConventionalMemory`（通常RAM） |

この1行が「メモリディスクリプタ」。

### 📦 UEFIの `EFI_MEMORY_DESCRIPTOR` の例（実物）

UEFI仕様では、メモリマップを構成するディスクリプタ構造体はこうなる：

```c
typedef struct {
  UINT32 Type;
  EFI_PHYSICAL_ADDRESS PhysicalStart;
  EFI_VIRTUAL_ADDRESS VirtualStart;
  UINT64 NumberOfPages;
  UINT64 Attribute;
} EFI_MEMORY_DESCRIPTOR;
```

#### 各フィールドの意味：

| フィールド名    | 意味                                       |
| --------------- | ------------------------------------------ |
| `Type`          | メモリの種類（後述）                       |
| `PhysicalStart` | このメモリ領域の先頭アドレス               |
| `NumberOfPages` | メモリのサイズ（4KB単位）                  |
| `Attribute`     | キャッシュ可能か、書き込み禁止かなどの属性 |

#### 🧩 Type（種類）の例

| 値                                      | 意味                     |
| --------------------------------------- | ------------------------ |
| `EfiConventionalMemory`                 | OSが使用できる通常メモリ |
| `EfiLoaderCode` / `EfiBootServicesData` | 起動時に使われる領域     |
| `EfiRuntimeServicesCode`                | UEFIランタイムサービス用 |
| `EfiReservedMemoryType`                 | 使用禁止の予約領域       |

### 🔧 どこで使われる？

OSやブートローダは、UEFIからこのメモリマップを受け取り、

* 「どこが空いているメモリか？」
* 「どこは使ってはいけないのか？」

を**メモリディスクリプタを使って判断**する。

### 🧠 例：自作OSのためのメモリマップ取得（UEFI環境）

UEFIアプリケーションではこんな風に取得する：

```c
UINTN mapSize = 0;
EFI_MEMORY_DESCRIPTOR* memoryMap = NULL;
UINTN mapKey, descriptorSize;
UINT32 descriptorVersion;

// 最初に必要なサイズを取得（バッファなしで呼び出す）
gBS->GetMemoryMap(&mapSize, memoryMap, &mapKey, &descriptorSize, &descriptorVersion);

// バッファを確保して再取得
memoryMap = AllocatePool(mapSize);
gBS->GetMemoryMap(&mapSize, memoryMap, &mapKey, &descriptorSize, &descriptorVersion);
```

→ ここで `memoryMap` に格納される各構造体が **メモリディスクリプタ**。

メモリマップとメモリディスクリプタの関係を図にすると次のようになる。
今回の場合、CSV出力したメモリマップはIndex=0~45の46行あった。
つまり、メモリマップの中にメモリディスクリプタは46個格納されているということになる。

![](/images/mikan-osbook-day2/memmap.drawio.png)


### ✅ まとめ

| 項目                       | 内容                                                                        |
| -------------------------- | --------------------------------------------------------------------------- |
| メモリディスクリプタとは？ | メモリの一領域に関する情報（アドレス、サイズ、種類など）を1つ分まとめたもの |
| どこで使う？               | OSやブートローダが、使ってよいメモリを判定するため                          |
| 含まれる情報               | 開始アドレス、ページ数、用途（種類）、属性など                              |
| 重要な理由                 | これがないと、誤ってデバイスや予約領域を上書きしてしまう恐れがある          |


## 糖衣構文 (Syntax Suger) とは？

**人間にとって読みやすく・書きやすくするための書き方**。
中身は同じでも、**見た目が甘く（=糖衣）、理解しやすい**ようにした記法を指す。

### 🍬 例：Pythonでの糖衣構文

```python
# 通常の書き方（内部でこう処理されている）
my_list = []
my_list.append(1)

# 糖衣構文（わかりやすい書き方）
my_list = [1]  # ← これが糖衣構文
```

### ✅ よくある糖衣構文の例

| 糖衣構文          | 展開される処理                   | 内容                   |
| ----------------- | -------------------------------- | ---------------------- |
| `for x in items:` | `iter()`と`next()`を使ったループ | イテレータ処理の簡略化 |
| `@decorator`      | 関数に別の関数を渡す             | デコレータの省略記法   |
| `a += b`          | `a = a + b`                      | 演算の省略記法         |


# その他

## Ansibleとは？

今回のMikanOSの開発環境構築には`Ansible`が用いられていた。
Ansible（アンシブル）は、**ITインフラの自動化・構成管理・アプリケーションデプロイ**を簡単に実現できるオープンソースのツール。
特に次のような特徴から、DevOpsやシステム管理の現場で非常に広く使われている。

### ✅ Ansibleの概要

| 項目         | 内容                                                                      |
| ------------ | ------------------------------------------------------------------------- |
| 開発元       | Red Hat（現在はIBM傘下）                                                  |
| 言語         | Pythonで実装されている                                                    |
| 管理対象     | Linux/Unix/Windowsサーバ、クラウド（AWS, Azure等）、ネットワーク機器 など |
| 接続方法     | SSH（またはWinRM）                                                        |
| インストール | 管理側の1台に入れるだけ（**エージェントレス**）                           |

### 🔧 何ができるのか？

Ansibleを使うことで、以下のタスクを自動化できる：

* パッケージのインストール・アップデート（例：`apt`, `yum`, `pip`）
* サービスの起動・停止・再起動
* ファイルのコピー、テンプレート展開
* ユーザーやグループの作成
* VMやクラウドリソースの作成・破棄
* アプリケーションのデプロイ

### 🧩 Ansibleの構成要素

#### 1. **Inventory（インベントリ）**

* 管理対象のホスト一覧を定義するファイル
* 例：

  ```ini
  [web]
  web1.example.com
  web2.example.com

  [db]
  db1.example.com
  ```

#### 2. **Playbook（プレイブック）**

* 実行したい処理手順（Play）の集合
* YAML形式で記述
* 例：

  ```yaml
  - hosts: web
    become: true
    tasks:
      - name: Install nginx
        apt:
          name: nginx
          state: present
  ```

#### 3. **Module（モジュール）**

* 処理の単位（ファイル操作、パッケージ管理、システム制御など）
* 1000以上の公式モジュールがあり、自作も可能

#### 4. **Role（ロール）**

* Playbook の再利用可能な部品化（ファイル構造で管理）
* 例：`webサーバ用ロール`, `MySQLセットアップロール`

#### 5. **Variables（変数）**

* 環境やホストによって異なる値を変数化できる

### 🚀 Ansibleの強みと利点

| 項目                   | 内容                                              |
| ---------------------- | ------------------------------------------------- |
| エージェントレス       | 管理対象にソフトを入れる必要なし（SSHで完結）     |
| 宣言的（Declarative）  | 「状態」を記述するだけでAnsibleが必要な操作を判断 |
| YAMLベースで読みやすい | コードが直感的で非エンジニアでも学びやすい        |
| 冪等性（idempotency）  | 同じPlaybookを何度実行しても結果は変わらない      |
| 拡張性                 | モジュール・フィルター・プラグインなど拡張可能    |

### 🧠 よくあるユースケース

| シナリオ      | 使用例                                   |
| ------------- | ---------------------------------------- |
| Webサーバ構築 | nginx, apache, PHP, SSL設定など          |
| DBサーバ構築  | PostgreSQLやMySQLのインストールと設定    |
| CI/CD自動化   | JenkinsやGitLabの構成管理やジョブ起動    |
| クラウド制御  | AWSのインスタンス作成やS3バケット作成    |
| OS共通設定    | firewall, sshd, cronなどのポリシー統一化 |

### 📘 実行例

```bash
ansible-playbook -i hosts site.yml
```

### ✅ まとめ

Ansibleは：

* **SSHベースのエージェントレス構成管理ツール**
* **Playbook（YAML）で直感的に構成・操作を記述**
* **インフラ構成、プロビジョニング、自動運用を統一的に管理**できる

# 次回

https://zenn.dev/kaiware0x/articles/mikan-osbook-day3

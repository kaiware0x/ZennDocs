---
title: "Releaseビルドのアプリがクラッシュしたときに原因ソース行を特定する方法"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "qt", "cmake", "windows"]
published: true
---

若干タイトル詐欺です。
**事前に設定しておけば**、あとからソース行を特定できます。


## 想定環境

- OS: Windows11
- Compiler: MSVC
- ビルド設定ツール: CMake
- デスクトップアプリ開発
- **管理者権限を取得可能**

## 要約

1. レジストリ設定でdmpファイル(プロセスダンプ)出力をON
2. ビルド設定でpdbファイル(デバッグシンボル)出力をON
4. Visual Studioにdmpファイルを読み込ませる
5. Releaseビルドでもクラッシュ箇所がソース行レベルで特定できる


## プロセスダンプの出力設定

参考：
https://jpwinsup.github.io/blog/2021/02/15/Performance/Hang_BSOD/AppDump/

Windows検索窓に`reg`と入力すると**レジストリエディター**が表示されるので起動する。

![](/images/analysis-dmp-with-windbg/2023-12-03-11-17-45.png)

`Ctrl+F`で検索ボックスを開き、`Windows Error Reporting`について検索する。
検索対象は**キーのみ**、また**完全一致のみ**を条件とする。

![](/images/analysis-dmp-with-windbg/2023-12-03-11-21-41.png)


`\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting`という項目にマッチするはず。
別の項目にマッチした場合は`F3`を押すことで次の検索結果に移動できる。

`Windows Error Reporting`を右クリック→新規→キーから`LocalDumps`を作成する。

![](/images/analysis-dmp-with-windbg/2023-12-03-12-32-37.png)


### Dumpファイルの出力先を設定

`LocalDumps`を右クリック→新規→展開可能な文字列を作成する。

![](/images/analysis-dmp-with-windbg/2023-12-03-12-33-08.png)

作成した値の名前を`DumpFolder`、値のデータを`C:\Dumps`(任意)に設定する。

![](/images/analysis-dmp-with-windbg/2023-12-03-11-43-52.png)

### Dumpファイルに出力する内容を設定

`LocalDumps`を右クリック→新規→DWORDを作成する。

![](/images/analysis-dmp-with-windbg/2023-12-03-12-33-52.png)

作成した値の名前を`DumpType`、値のデータを`2`に設定する。
`2`はフルダンプを意味しており、出来得る限りすべての情報をDumpファイルに出力する。

![](/images/analysis-dmp-with-windbg/2023-12-03-11-51-09.png)

### Dumpファイル数の最大数を設定

`LocalDumps`を右クリック→新規→DWORDを作成する。

![](/images/analysis-dmp-with-windbg/2023-12-03-12-34-22.png)

作成した値の名前を`DumpCount`、値のデータを`10`(1以上)に設定する。
Dumpファイルは1つが数GBになることも多いので、ストレージ容量に応じて値を変更することを推奨。

![](/images/analysis-dmp-with-windbg/2023-12-03-11-54-06.png)

### 最終確認

下図のような値になっているか確認する。

![](/images/analysis-dmp-with-windbg/2023-12-03-12-34-47.png)


## ビルド設定でPDBファイル出力をONにする

参考：
https://stackoverflow.com/questions/28178978/how-to-generate-pdb-files-for-release-build-with-cmake-flags

`PDB`ファイルはデバッグシンボルと呼ばれ、バイナリとソースコードのファイル名や行番号やシンボル情報 (変数名等) の対応関係を保持している。
デバッグ実行時、`PDB`ファイルを参照してプログラムのステップ実行を可能にしている。

Relaseビルドでは通常`PDB`ファイルは出力されないが、CMakeスクリプトに以下の設定を記述することで出力可能。

```cmake
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi" CACHE STRING "" FORCE)
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF" CACHE STRING "" FORCE)
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF" CACHE STRING "" FORCE)
set(CMAKE_STATIC_LINKER_FLAGS_RELEASE "${CMAKE_STATIC_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF" CACHE STRING "" FORCE)
```

:::message alert
ユーザリリース時には上記設定はOFFにすること！
:::


## サンプルプログラムを用意

Qtで作成。
ボタンを押したらNullアクセスでクラッシュする。

```cpp
#include <QApplication>
#include <QPushButton>

int main(int argc, char* argv[])
{
    QApplication a(argc, argv);

    auto button = new QPushButton("Push");

    QObject::connect(button, &QPushButton::clicked, []() {
        QWidget* widget{nullptr};
        widget->size();
    });

    button->show();

    return a.exec();
}
```
![](/images/analysis-dmp-with-windbg/2023-12-03-13-19-03.png)

ビルドする。
出力フォルダにPDBファイルが生成されていることを確認する。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-20-24.png)

プログラムを実行し、ボタンを押してクラッシュさせる。

レジストリエディターで設定した`C:\Dumps`フォルダにDumpファイルが生成されている。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-21-59.png)


## Visual StudioでDumpファイル解析


Visual Studioを起動し、「コードなしで続行」をクリックする。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-31-07.png)

ファイル→開く→ファイル を選択し、Dumpファイルを開く。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-31-39.png)

Dumpファイルの概要が表示される。
具体的にソースコードのどの行でクラッシュしたかを表示するため、**シンボルパスの設定**をクリックする。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-33-51.png)

今回はQtのバイナリが配置されているフォルダと、先程ビルドしたプログラムとPDBファイルがあるフォルダを追加する。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-35-24.png)

**ネイティブのみでデバッグ**をクリックする。
![](/images/analysis-dmp-with-windbg/2023-12-03-13-36-50.png)

:::message
マネージドや混合はおそらくC++/CLIを利用している場合にのみ選択する
:::


プログラムがクラッシュした箇所が**ソースコードの行レベルで**表示される。

![](/images/analysis-dmp-with-windbg/2023-12-03-13-38-54.png)


## 感想

Releaseビルドで実行中に突然クラッシュしてもクラッシュ箇所を特定できるので便利。
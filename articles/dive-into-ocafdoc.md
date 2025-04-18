---
title: "OCAF Documentファイルの可視化方法 [OpenCASCADE]"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["OpenCASCADE", "cpp", "OCAF"]
published: true
---

## はじめに

こんにちは、**kaiware**です。

**OpenCASCADE(以下OCC)** というC++のCADカーネルライブラリについての話です。

OCCには**OCAF(OpenCASCADE Application Framework)** というモジュールがあります。
素のOCCに3D形状のBoolean演算やサイズ編集などの機能はありますが、それをアプリケーションとして組み立てるためにはOCAFを使うと便利です。

OCAFを使うことで以下のことが簡単(??)にできます：

- 3D形状の作成・変更の履歴管理
- ストレージへの保存(永続化)
- Undo/Redo
- Copy/Paste

これらの機能を使うには専用の**Document**クラスと**Application**クラスを用いる必要があります。

本記事では、この**Documentをファイルに保存したあとに可視化する方法**を説明します。
これができると、OCAFを使っているときに遭遇するよくわからない挙動をデバッグできるようになるかもしれません。

## 環境

- Windows11
- OpenCASCADE 7.8.0

## OpenCASCADEのインストール

今回はソースからビルドはせず、ビルド済みバイナリをインストールする。

ここから OpenCASCADE 7.8.0 をインストールする。
https://dev.opencascade.org/release/previous#node-87560

![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-2.png)

7.8.0を使う理由は特にないので、別のバージョンでも大丈夫とは思う。

またOCC本体のインストール先と3rd-partyライブラリの置き場の注意書きがあるので、この通りに配置する。

![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-3.png)

## テスト用OCAF Documentファイルの作成

今回はフリーのCAD Viewerである **CAD Assistant** を使ってテスト用のDocumentファイルを作る。
ちなみにCAD AssistantはOCCと開発元が同じ。

ここからインストールする。
https://www.opencascade.com/products/cad-assistant/

![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-4.png)

インストールが完了すると、サンプルのCADデータ(STEPファイル)も配置されている。
ここでは`as1-oc-214.stp`というファイルを使う。
(ファイルパス例： `"C:/Program Files/CAD Assistant/resources/samples/as1-oc-214.stp`)

CAD Assistantで開き、Saveボタンを押す。

![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-5.png)

適当な場所へ移動し、保存フォーマットを **XBF** に変更して保存する。
XBFがOCAF Documentのファイル形式の1つである。
![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-6.png)

## テストハーネスの起動

OCCには`DRAW Test Harness`という簡単な動作テスト環境が用意されている。
参考：https://dev.opencascade.org/doc/overview/html/occt_user_guides__test_harness.html

インストールフォルダに`DRAWEXE.exe`という実行ファイルがあり、これが本体。
ただこれを直接起動すると環境変数がセットされていないので、同じ階層にある`draw.bat`を起動する。

▼ 起動直後
![起動直後](/images/dive-into-ocafdoc/dive-into-ocafdoc.png)

次のコマンドを入力する。(Plugin Load ALLの意味)
```
pload ALL
```

▼ 実行後：Drawウィンドウのメニューバーの種類が増えている
![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-1.png)

次のコマンドで先ほど保存したXBFファイルを開き、`doc`変数へ格納する。

```
Open D:/tmp_ocaf/as1-oc-214.xbf doc
```

次のコマンドで`doc`変数を可視化する。

```
DFBrowse doc
```

▼ 実行後：browser_docというウィンドウが起動する
![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-7.png)

`browser_doc`と書かれた箇所をダブルクリックすると、OCAF Document内のツリー構造が見れる。

![alt text](/images/dive-into-ocafdoc/dive-into-ocafdoc-8.png)

中身を簡単に説明する。

- `0:doc`
    - `Root Label`と呼ばれるツリーの根の情報を表している。
        - OCAFはLabelという概念を用いてツリー構造を構築する。(`TDF_Label`クラスを参照。)
    - 下に書かれている黒文字はLabelが持つ属性情報を表している。(`TDF_Attribute`クラスを参照)
- `0:1`
    - `Main Label`と呼ばれる様々な情報の親Label。
- `0:1:1 Shapes`
    - 形状データが格納されている。(`TNaming_NamedShape`)
    - また、特定のShapeへの参照リンクが張られていることもある。(`Shape Instance Link`)
- `0:1:2 Colors`
    - Document内で使われる色情報が一元管理されている。(`XCAFDoc_Color`)
- `0:1:3`以降は自分もよくわからないので省略

### Shapes, Colors詳細


- アセンブリ全体は4つのアセンブリと部品から構成されている
    - as1
        - rod
        - bracket
        - bracket
        - plate

以下plateを例にとる

![alt text](/images/dive-into-ocafdoc/image24.png)

- `Shape Instance Link` という属性を持っている
    - これが実ShapeへのLink
        - Entry 0:1:1:9
    - `plate 1` ラベル自体はShapeを持っていない

![alt text](/images/dive-into-ocafdoc/image25.png)

- Entry 0:1:1:9 を見ると、`Surface Color Link` という属性を持っている
    - Entry 0:1:2:3
- また、 `TNaming_NamedShape` という実Shapeを管理する属性を持っている

![alt text](/images/dive-into-ocafdoc/image26.png)

- Entry 0:1:2:3 を見ると、実際の色情報(黄色？黄緑？)が格納されている

## まとめ

OCCの概念難しすぎるだろ！

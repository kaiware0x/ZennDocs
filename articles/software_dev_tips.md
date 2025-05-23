---
title: "あなたのソフトウェア開発効率を改善するちょっとしたTips集"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "python", "tech", "vscode", "qtcreator", "editor"]
published: true
---

こんにちは、**kaiware**です。

この記事では、私が日々のプログラミングで気をつけているポイントや、効率を上げるためのテクニックをまとめてみました。C++（QtCreator）とPython（VSCode）を想定していますが、**他の言語やツールでも応用可能**な内容です。ぜひ参考にしてみてください！

**またオススメのTipsがあれば、ぜひコメントで教えてください！**

※ この記事はベースは人間が作成し、体裁を整えたり情報を追加したりなどはChatGPTで行いました。

---

## 1. 見やすいコードを書く工夫

### シンタックスハイライトのカスタマイズ

**例**:

- デフォルトでは薄いグレーのコメントを、目立つ緑色に変更する。
- 複数の種類のリテラル（文字列、数値）を異なる色で表示する。

→ **「目で見てコードの構造が直感的に理解できる」** を目指すことが重要です。

私は以下のように色分けしています(QtCreatorでの例)。Visual Studioライクな色分けが好きです。

| 項目            | 色         |
| --------------- | ---------- |
| 定義全般        | 薄ピンク   |
| Class利用       | 翡翠       |
| 関数利用        | 黄色       |
| Local変数       | 水色       |
| Keyword         | 青         |
| Static定数利用  | 紫         |
| 名前空間        | 濃いピンク |
| 通常コメント    | 灰色       |
| Doxygenコメント | 緑         |

![](/images/software_dev_tips/2024-12-28-21-19-14.png)

### フォントのカスタマイズ

見やすいフォントを使うと視認性が向上し、ソースも読みやすくなります(当社比)。

おすすめフォントですが、`HackGen`などで有名な**tawaraさん**が作成されたフォントなら間違い無いと思っています。
このリポジトリから好みのものを選んでインストールしてください。

https://github.com/yuru7

---

## 2. コード設計と命名規則

### 変数名の付け方

**NG例**:

```cpp
int a = 5; // a とは何か分からない
```

**OK例**:

```cpp
int itemCount = 5; // 明確で意味が伝わる
```

### 関数名の付け方

**NG例**:

```cpp
int total(); // 動詞が無い (Getterなどシンプルな関数ならアリ)
```

**OK例**:

```cpp
int calculateTotal(); // 動詞が先頭で明確
```

### 変数の長さと登場頻度・寿命

**変数のスコープはできるだけ短く**保つのが基本です。また、**変数名の長さはその寿命や登場頻度に比例**させると良いでしょう。
ここは定量化が難しいところです。たくさん書いて感覚を身に着けてください。
下図は個人的な長さの指標です。

![](/images/software_dev_tips/scope_and_freq.drawio.png)

:::message alert
**そもそも寿命が長い変数はできるだけ作らないようにしましょう。**
:::

**例**:

- 1行程度の短いスコープなら`i`のような極めて短い名前を使用して良い。
- グローバル変数や長寿命の変数なら、`globalSettings`のように具体的で分かりやすい名前を付ける。



---

## 3. 作業効率を高めるTips

### エディタのショートカットを覚える

**マウスは💩です。** 💩に触りたいですか？触りたいなら何も言いません。

一般的には手をキーボードから離さずに作業することで、コードを書くスピードが飛躍的に上がります。

下表は**VSCodeのおすすめキーボードショートカット**です。
知名度が低そうなものをいくつか選びました。他にも色々あるので探してみてください。

| ショートカット     | 効果                                  |
| ------------------ | ------------------------------------- |
| Ctrl + @           | ターミナルを開く                      |
| Ctrl + J           | 下部パネルを開く・閉じる (トグル)     |
| Ctrl + K, Ctrl + 0 | コードブロックすべて折り畳み          |
| Ctrl + K, Ctrl + J | コードブロックすべて展開              |
| Ctrl + K, Ctrl + L | コードブロック折り畳み・展開 (トグル) |
| Ctrl + P           | ファイル検索                          |
| Ctrl + Shift + L   | 選択した単語のすべてを同時編集        |

### スニペットの活用

**頻繁に使うコードのテンプレートはスニペットに登録**しておきましょう。例えば、定型的なコメントブロックや関数の雛形をすぐに呼び出せるようにすると便利です。

こちらの記事が参考になります。

https://qiita.com/12345/items/97ba616d530b4f692c97

私は以下のようなスニペットを登録して、`cc`と押下するとコメントブロックを出せるようにしています。

```json
"Python comment block": {
    "scope": "python",
    "prefix": "cc",
    "body": [
        "##################################################",
        "### $1",
        "##################################################",
    ],
    "description": "Python comment block"
},
```

---

## 4. 安定したコードを書くために

### リテラル文字列は定数化する

**リテラル文字列（固定文字列）はミスを引き起こしやすい**ため、定数に置き換えるのがおすすめです。定数にすれば入力補完が効き、タイポを防ぐことができます。

**NG例：**

```cpp
map<string, Property> proparty_map;

/// プロパティをセット...

/// プロパティの取得
const auto val1 = proparty_map["key1"];
const auto val2 = proparty_map["kay2"]; ///< 本当は"key2"だがタイポに気付かない！
```

**OK例：**

```cpp
map<string, Property> proparty_map;
const string KEY1 = "key1";
const string KEY2 = "key2";

/// プロパティをセット...

/// プロパティの取得
const auto val1 = proparty_map[KEY1];
const auto val2 = proparty_map[KEY2]; ///< 入力補完が効く！
```

### コンパイル時定数の活用

コンパイル時に確定できる定数は可能な限り利用しましょう。実行時の負荷を軽減し、パフォーマンスを向上させることができます。

**例**:

```cpp
// OK: constexprでコンパイル時に定義
constexpr int BUFFER_SIZE = 1024;
char buffer[BUFFER_SIZE];
```

---

## 5. テストを書く習慣

コードの信頼性を上げるためには、テストを書くことが不可欠です。特に、動きが頻繁に変わる環境では、テストがバグからプロダクトを守る最良の盾になります。

以下のユニットテストのフレームワークのうち、使いやすいものを使えば良いと思います。

- C++の場合：
    - CTest
    - GoogleTest
    - QTest
- Python：
    - pytest
    - unittest

---

## 6. チーム開発で役立つ工夫

### Git コミットメッセージの管理

**コミットメッセージには一貫したフォーマットを採用**すると、履歴がわかりやすくなります。ただし種類が多すぎると混乱するので、私はシンプルに以下のみで運用しています：

- **💎feat**: 新機能の追加 (既存の動作が変化する変更)
- **💡refac**: コードのリファクタリング (既存の動作が変化しない変更)
- **🔨fix**: バグ修正
- **📝docs**: ドキュメントの変更
- 🏗️**chore:** その他ビルドスクリプトなどの変更

**NG例**:

```
Updated files
```

**OK例**:

```
💎feat: Implement user authentication
🔨fix: Correct buffer overflow in login handler
```

上記のようなルールを導入すれば、履歴をもとにPythonなどで**半自動的にChangeLogを生成**することも可能です。

次のコマンドを打つと、Gitのコミットメッセージをテキストファイルに書き出せます。

例：2024/1/25以降のコミットメッセージを`commit_log.log`ファイルに書き出す

```bash
git log --no-merges --oneline --all --reverse --date=iso --after="2024/1/25" --pretty=format:"[%ad] [%cn] %s" > commit_log.log
```

書き出したログ・ファイルをPythonで処理すると、Markdown形式のChangeLogが生成できます。

以下、Pythonスクリプトです。

@[gist](https://gist.github.com/kaiware0x/2e64f549b86b11ca546dc1d5da87b72e)

---

## 7. デバッグの効率化

### ログの活用

デバッグを効率的に進めるために、適切なログを出力することが重要です。

- **ログレベルを使い分ける**：DEBUG、INFO、WARN、ERRORなど、メッセージの重要度を明確に区別します。
- **ロギングライブラリの使用**：C++なら`spdlog`、Pythonなら標準ライブラリの`logging`を活用することで、ログ出力の管理が簡単になります。
    - ※ 使いやすいライブラリなら何でも良いと思います

**NG例**:

```cpp
/// NG: どこのエラー？ 重要度は？
std::cout << "Error occurred!" << std::endl;
```

**OK例**:

```cpp
#include <spdlog/spdlog.h>
/// OK: エラー箇所と重要度がわかる
spdlog::error("Error occurred in function {}", __FUNCTION__);
```

---

## 8. パフォーマンスの最適化

### メモリ管理に注意

特にC++を使用する場合、動的メモリ管理に注意が必要です。

- **スマートポインタを活用する**：手動で`delete`を呼ぶ必要がなくなり、メモリリークを防止します。
- **スタックを優先する**：可能であれば、ヒープよりもスタックを使用することでメモリアクセスが速くなります。

### プロファイリングツールの活用

コードのパフォーマンスを測定するためにプロファイリングツールを活用しましょう。以下が代表的なツールです：

- **C++**: Valgrind、gprof
- **Python**: cProfile、line_profiler

また以下のツールならば言語によりません(CPUによります)。**無料**なのでぜひ使ってみてください。

- **AMD製CPU**: AMD uProf Profiler
- **Intel製CPU**: Intel VTune Profiler

---

## 9. コード品質の維持

### 静的解析ツールや自動フォーマッターを導入する

コードのバグやスタイル違反を事前に防ぐために、静的解析ツールを積極的に利用しましょう。

- **C++**: clang-format
- **Python**: pylint、flake8

これらのツールをCI（継続的インテグレーション）に組み込むと、コードの品質を自動でチェックできるようになります。
またVSCodeなどの高機能エディタでは、**ファイルの保存時に自動フォーマット**をかけることもできます。設定から`formatOnSave`で検索してみてください。

## 10. 学び続ける姿勢

### コミュニティに参加する

新しいアイデアや知識を得るために、プログラミングコミュニティに参加してみましょう。

- **オンラインフォーラム**: Stack Overflow、Reddit
- **SNS**: Twitterで開発者をフォロー
- **イベント**: オンライン・オフラインのハッカソンやミートアップに参加

### 他の言語やツールにも挑戦

普段使っている言語以外にも挑戦することで、新たな視点や効率的な手法を学べます。例えば、RustやGoは近年注目を集めており、学習する価値があります。

### とにかく書け！

言語の文法や局所的なコードの綺麗さについて、**考えて書いているうちは甘い**です。(設計はちゃんと考えよう)
手が自動で動くようになるまで、プログラムをひたすら**書いて書いて書きまくってください。**

:::message alert
**脳のワーキングメモリはもっと価値のあることに使いましょう。**
:::

---

## おわりに

この記事で紹介したポイントを実践すれば、コードの品質向上だけでなく、作業効率もアップするはずです。ぜひ日々のプログラミングに取り入れてみてください！

**あなたのおすすめのTipsも、ぜひ教えてください！**

---
title: "[C++] Visitorパターン with std::variant, std::visit"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [cpp,tech,入門,]
published: true
---

## 初めに

本記事は以下の本の内容を要約したもの。

https://www.oreilly.co.jp/books/9784814400454/


## `std::variant`とは

C++17以降で使用可能な型。
複数の型のうちのいずれか一つを保持できるコンテナ。
型安全なユニオンとして機能し、多態性を実現する一つの方法となる。

```cpp
// 保持できる型をテンプレート引数として指定する。
std::variant<int, double, std::string> var;

// 代入した型に応じて内部状態が変化する。
var = 42;  // int
var = 3.14;  // double
var = "hello";  // std::string

/// int型の値を取得.
/// varに格納されている値がint以外の場合は例外が飛ぶ.
try {
    const int i = std::get<int>(var);
    std::cout << "var contains int: " << i << '\n';
}
catch (std::bad_variant_access e) {
    std::cout << e.what() << '\n';
}

/// int型の値のポインタを取得.
/// varに格納されている値がint以外の場合はnullptrになる.
if (const int *i_ptr = std::get_if<int>(&var)) {
    std::cout << "var contains int: " << *i_ptr << '\n';
}
else {
    std::cout << "var does not contains int" << '\n';
}

if (std::holds_alternative<int>(var)) {
    // varはintを保持している
}

```

## `std::visit`とは

C++17以降で使用可能な関数。
`std::variant`に保持されている値に対して柔軟な操作を行うためのメカニズム。
Visitorパターンを簡単に実装できる。

```cpp
/// 関数オブジェクト
struct Print {
    Print() { std::cout << __func__ << std::endl; }
    void operator()(const int &value) const { std::cout << "int: " << value << '\n'; }
    void operator()(const double &value) const { std::cout << "double: " << value << '\n'; }
    void operator()(const std::string &value) const { std::cout << "string: " << value << '\n'; }
};

std::variant<int, double, std::string> var;

var = "hello";  // std::string

/// varに現在格納されている値から適切なoperator()が呼ばれる.
std::visit(Print{}, var);
```

## Visitorパターンの実装 (古典的)

リポジトリはこちら。

https://github.com/kaiware0x/CppSoftwareDesign/tree/main/04-1_Visitor_Classical

デザインパターンの一つであり、**オブジェクトの構造**とその上で行う**操作**を分離する目的で使用される。
オブジェクトの構造が頻繁に変更されないが、新しい操作を追加する必要がある場合に有用。


## Visitorパターンの実装 (`std::variant`と`std::visit`を用いて)

リポジトリはこちら。

https://github.com/kaiware0x/CppSoftwareDesign/tree/main/04-2_Visitor_stdvariant

### Pros

- クラス間の参照が循環しない
- 仮想関数呼び出しが発生しない
  - コンパイラの最適化がよく効き、性能が良い
- ポインタが必要ない。すべて値ベースで実装できる
- 処理のオープンセット(追加しやすさ)を実現する

### Cons

- 型のクローズセット(追加しにくさ)に対応する必要がある
- `std::variant`に持たせる型の1つのサイズが大きいと、`std::variant`全体のサイズが大きくなってしまう
  - `Proxy`オブジェクトや`Bridge`パターンを用いると緩和できる


---
title: "[C++] ポインタ型変数でも関数呼び出し時には実体参照渡ししよう"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp"]
published: true
---


こんにちは、**kaiware**です。

C++での話になります。タイトル通りです。

以下のようにnewで生成したインスタンスを外部の関数に渡す場合を考えます。

その際、インスタンスを利用する側の関数は参照で受け取るようにすれば
Nullチェックのダブりを防げて効率がよいと思い、活用しています。

:::message alert
**下記はあくまで例です。そもそも生ポインタを使わずスマートポインタを活用しましょう。もっと言えばポインタではなくスタック変数を使いましょう。**
:::

```cpp
#include <iostream>

struct MyClass {
    int x;
    int y;
}

void func_call_by_pointer(MyClass* ptr){
    /// ptrのNullチェックをしたくなるが、呼び出し元でもされている。
    /// → 二重チェックになり効率が悪い。
    if(!ptr){
        return;
    }

    std::cout << ref->x << ref->y << std::endl;
}

void func_call_by_reference(MyClass& ref){
    /// 参照で受け取ればこの関数内でNullチェックの必要がない
    /// → 呼び出し元でNullチェックされている前提で処理をかける！
    /// → 関数内で意図せずDeleteされる危険性も小さくできる！

    std::cout << ref.x << ref.y << std::endl;
}

int main(){
    auto mc = new MyClass{1, 2};

    /// 呼び出す前にNullチェックしたくなるのがC++プログラマの性
    if(mc) {
        func_call_by_pointer(mc);
        func_call_by_reference(*mc); /// 実体にアクセスして関数呼び出し
    }
    return 0;
}
```

ご意見ありましたらぜひコメントお願いします！

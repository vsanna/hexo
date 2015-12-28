title: "C++ の世界へ入門 (双方向リスト実装練習 + Boost.Test)"
date: 2015-12-28 23:49:00
tags:
- necojackarc
- C++
- データ構造
---

C++ の世界へ入門してみることにしました。

## Motivation
動機は、

- 競技プログラミング/プログラミングコンテストで使える
- Ruby の Native Extensions も書ける
- 爆速

って感じです。

趣味と実益を兼ねるし、なんか面白そうだし、俺も C++ 書くわ、みたいなノリです。
あと、C++er ってカッコイイですよね。憧れちゃう。

## Boost.Test
競プロで使うことがあるかは微妙ですが、ユニットテストも書いてみようと思い、Boost.Test を導入してみました。

C++ 関連のテスティングフレームワークを調べてみたところ、

- [Boost.Test](http://www.boost.org/doc/libs/1_60_0/libs/test/doc/html/index.html)
- [CppUTest](http://cpputest.github.io/)
- [Google Test](https://github.com/google/googletest)

あたりが良さそうだなとなんとなく思いました。
今回は Boost.Test を試しに使ってみました。

## Linked List
C++ の練習 + 基礎力向上のため、データ構造の基本中の基本である双方向リストを実装してみました。

```C++
template <typename T>
class LinkedList {
    struct Node {
        T data_;
        Node *prev_;
        Node *next_;

        Node() {
            prev_ = next_ = this;
        }

        Node(T data, Node *prev, Node *next) {
            data_ = data;
            prev_ = prev;
            next_ = next;
        }
    };
    Node *sentinel_;
    Node *cursor_;

public:
    LinkedList() {
        sentinel_ = cursor_ = new Node();
    }

    ~LinkedList() {
        purge();
        delete sentinel_;
    }

    bool isEmpty() {
        return sentinel_->next_ == sentinel_;
    }

    bool hasNext() {
        return !(isEmpty() || cursor_->next_ == sentinel_);
    }

    bool contains(T data) {
        return getNodeFromData(data) != sentinel_;
    }

    T next() {
        cursor_ = cursor_->next_;
        return cursor_->data_;
    }

    void rewind() {
        cursor_ = sentinel_;
    }

    T get(int index) {
        return getNodeFromIndex(index)->data_;
    }

    T getCurrentData() {
        return cursor_->data_;
    }

    // Add the element at the top of the list
    void prepend(T data) {
        Node *node = new Node(data, sentinel_, sentinel_->next_);
        sentinel_->next_ = sentinel_->next_->prev_ = node;
    }

    // Add the element at the bottom of the list
    void append(T data) {
        Node *node = new Node(data, sentinel_->prev_, sentinel_);
        sentinel_->prev_ = sentinel_->prev_->next_ = node;
    }

    void insert(int index, T data) {
        Node *target_node = getNodeFromIndex(index);
        Node *inserting_node = new Node(data, target_node->prev_, target_node);
        target_node->prev_ = target_node->prev_->next_ = inserting_node;
    }

    void insertBeforeCurrentData(T data) {
        Node *node = new Node(data, cursor_->prev_, cursor_);
        cursor_->prev_ = cursor_->prev_->next_ = node;
    }

    void insertAfterCurrentData(T data) {
        Node *node = new Node(data, cursor_, cursor_->next_);
        cursor_->next_ = cursor_->next_->prev_ = node;
    }

    void replace(int index, T data) {
        getNodeFromIndex(index)->data_ = data;
    }

    void replaceCurrentData(T data) {
        cursor_->data_ = data;
    }

    void remove(int index) {
        Node *target_node = getNodeFromIndex(index);
        target_node->prev_->next_ = target_node->next_;
        target_node->next_->prev_ = target_node->prev_;
        delete target_node;
    }

    void removeCurrentData() {
        cursor_->prev_->next_ = cursor_->next_;
        cursor_->next_->prev_ = cursor_->prev_;

        Node *tmp = cursor_;
        cursor_ = tmp->next_;
        delete tmp;
    }

    void purge() {
        cursor_ = sentinel_->next_;
        while (!isEmpty()) {
            removeCurrentData();
        }
    }

private:
    Node *getNodeFromIndex(int index) {
        Node *node = sentinel_->next_;
        for (int i = 0; i < index; i++) {
            node = node->next_;
        }
        return node;
    }

    Node *getNodeFromData(T data) {
        Node *node = sentinel_->next_;
        while (node != sentinel_) {
            if (node->data_ == data) break;
            node = node->next_;
        }
        return node;
    }
};
```

演算子のオーバーロード機能を使って`list[n]`みたいに操作できるようにすれば良かったかなーと思います。

雑なテストはこちら。
全部のメンバ関数呼び出してれば良いっしょなテストです。

```C++
#define BOOST_TEST_MODULE Linked List Test
#include <boost/test/included/unit_test.hpp>
#include "../src/linked_list.hpp"

struct Fixture {
    LinkedList<std::string> list;

    Fixture() {
        list.append("Hello");
        list.append("World");
    }
};

BOOST_AUTO_TEST_CASE(contains_item) {
    Fixture f;
    BOOST_TEST(f.list.contains("Hello"));
    BOOST_TEST(!f.list.contains("hello"));
}

BOOST_AUTO_TEST_CASE(get_item) {
    Fixture f;
    BOOST_TEST(f.list.get(1) == "World");

    f.list.next();
    BOOST_TEST(f.list.getCurrentData() == "Hello");
}

BOOST_AUTO_TEST_CASE(purge_list) {
    Fixture f;
    BOOST_TEST(!f.list.isEmpty());

    f.list.purge();
    BOOST_TEST(f.list.isEmpty());
}

BOOST_AUTO_TEST_CASE(show_list_items) {
    Fixture f;
    std::string str = "";

    while (f.list.hasNext()) {
        str += f.list.next();
    }
    BOOST_TEST(str == "HelloWorld");

    f.list.rewind();
    BOOST_TEST(f.list.hasNext());
}

BOOST_AUTO_TEST_CASE(prepend_item) {
    Fixture f;
    f.list.prepend("!");
    BOOST_TEST(f.list.get(0) == "!");
}

BOOST_AUTO_TEST_CASE(append_item) {
    Fixture f;
    f.list.append("!");
    BOOST_TEST(f.list.get(2) == "!");
}

BOOST_AUTO_TEST_CASE(insert_item) {
    Fixture f;
    f.list.insert(2, "!");

    f.list.next();
    f.list.insertBeforeCurrentData("^");
    f.list.insertAfterCurrentData("_");

    f.list.rewind();
    std::string str = "";
    while (f.list.hasNext()) {
        str += f.list.next();
    }
    BOOST_TEST("^Hello_World!");
}

BOOST_AUTO_TEST_CASE(replace_item) {
    Fixture f;
    f.list.replace(1, "Boost");

    f.list.next();
    f.list.replaceCurrentData("Hi");

    f.list.rewind();
    std::string str = "";
    while (f.list.hasNext()) {
        str += f.list.next();
    }
    BOOST_TEST("HiBoost");
}

BOOST_AUTO_TEST_CASE(remove_item) {
    Fixture f;
    f.list.remove(1);

    std::string str = "";
    while (f.list.hasNext()) {
        str += f.list.next();
    }
    BOOST_TEST("Hello");

    f.list.removeCurrentData();
    BOOST_TEST(f.list.isEmpty());
}
```

RSpec 並に見やすいテスト結果の出力が欲しいなと思いました。
単に使いこなせてないだけのような気もします。

オススメのテスティングフレームワークがあったら教えて下さい！

## Tasks
- 演算子オーバーロードの導入
- C++ のコーディング規約に従ってない感じしかしない
- テンプレートクラスのヘッダー実装分離がうまくできなかった
    - テンプレートクラス内の構造体を戻り値とする Private メンバ関数でハマった


## Tips
[thinca/vim-quickrun](https://github.com/thinca/vim-quickrun)を入れておくとかなり快適です。

- [quickrun.vim について語る](http://d.hatena.ne.jp/osyo-manga/20130311/1363012363)

こちらのブログを参考に設定しました。

## Impression
C++ デビューしましたが、予想以上に面白いです。
[高等黒魔術言語](http://ja.uncyclopedia.info/wiki/C%2B%2B)との噂もありますが、高等黒魔術言語だからこそ面白いように感じます。

良い感じの言葉に直すと、

> C++は主要な開発言語であり、Googleのオープンソースプロジェクトでも数多く使われています。 C++プログラマであればだれでも知っているように、C++には強力な機能がたくさんあります。ところが、その力ゆえに複雑でもあります。C++で書かれたコードはバグを生みやすく、読みにくく、メンテナンスがしにくくなるおそれがあるのです。

> <cite>[Google C++スタイルガイド 日本語訳](http://www.textdrop.net/google-styleguide-ja/cppguide.xml)</cite>

ということです。流石 Google 先生。

C++ の世界へいざ行かん！

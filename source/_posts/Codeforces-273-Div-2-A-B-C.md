title: "Codeforces #273 Div.2 A B C"
date: 2015-05-02 13:33:42
tags:
- necojackarc
- Codeforces
- Scala
---

- [個人ブログ](http://necojackarc.hatenablog.com/entry/2015/04/30/023443)から転載

---

A, B, C と解いてみました。Cは正直解けませんでした。降参してググって、回答みて感動しました……そんなの気がつかないよ！！

## A. Initial Bet
問題はこんな感じ。正直、英語力がヤバい。辛い。

> 全員に同じ枚数のコインを与えてゲームを始める。ゲームをするとコインのやりとりが発生する。最終的なコインの状態から、最初に与えられたコインは何枚だったのか求めよ。おかしな状態だったら-1を出力せよ。

超楽勝ですね。全員の枚数を足して、人数で割ってあまりがない場合の商が答えです。
全員0点だった場合も-1にしないといけないそうです。1度提出してから気がつきました。

```scala
import java.util.Scanner

object R273_InitialBet {
  def main(args:Array[String]) = {
    val in = new Scanner(System.in).nextLine().split(" ").map(_.toInt)
    val sum = in.sum
    val len = in.length

    println(sum match {
      case 0 => -1
      case _ if sum % len == 0 => sum / len
      case _ => -1
    })
  }
}
```

## B. Random Teams
英語力の足りなさを感じる。

> n人の参加者をmチームに分ける。各チームは最低1人。同じチームの人は友人になる。友人の組み合わせの最小と最大を求めよ。

どういうときに最小になるのか、どういうときに最大になるのかを考えてみました。

- 最小: 各チームの人数差を高々1にする
- 最大: 可能な限り大人数のチームを作る (1チーム以外はメンバー1人のチームにする)

正直、完全に勘です。実装して提出してから合ってるか考えましょう。

```scala
import java.util.Scanner

object R273_RandomTeams {
  def main(args: Array[String]) = {
    val in = new Scanner(System.in).nextLine().split(" ").map(_.toInt)
    val (n, m) = (in(0), in(1))
    val (p, q) = (n / m, n % m)

    val min = combination2(p) * (m - q) + combination2(p + 1) * q
    val max = combination2(n - m + 1)
    println(min + " " + max)
   }

  def combination2(n: Long): Long = {
    (n * (n - 1)) / 2
  }
}
```

なぜか合っていました。

ポイントは組み合わせ (combination) の求め方です。
最初はバカ正直に実装してたんですが、与えられる入力が10^9と結構大きいため、組み合わせを求める公式通りに計算したら時間オーバーします。

そのため、常に nC2 となることを利用して、式を展開しておきます。あと、Int では なく Long を使い桁あふれを防ぎます。

## C. Table Decorations
これは綺麗な解法思いつきませんでした。ググっちゃいました。

> 赤、緑、青の風船が与えられる。風船3つを使って机をデコレーションせよ。ただし、同じ色の風船のみの机は作ってはならない。最大いくつの机をデコレーションできるか求めよ。

これ、どうやら、

1. 一番大きいものが、それ以外の2つの和の2倍以上のとき
2. それ以外のとき

の2パターンの場合わけでいけるっぽいです。詳細な説明は割愛しますが、1の場合は一番大きいものから常に2つ取る方法が最大になり、それ以外のときは単に合計を3で割れば良いです。

以下、ヒント

```
RRRRRRGGGG
GGGGGGGBBB
BBBBBBBBBB
```

一番大きいものが、それ以外の2つの和の2倍以上のときって、全ての列にとある色が出現しちゃう状況ってことです。

```scala
import java.util.Scanner

object R273_TableDecorations {
  def main(args: Array[String]) = {
    val in = new Scanner(System.in).nextLine().split(" ").map(_.toLong).sorted
    println(if ((in(0) + in(1)) * 2 <= in(2)) in.take(2).sum else in.sum / 3)
  }
}
```

こんなの気がつかないよ……！

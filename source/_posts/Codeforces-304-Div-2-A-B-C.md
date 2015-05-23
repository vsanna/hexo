title: "Codeforces #304 (Div.2) A, B, C"
date: 2015-05-24 02:03:38
tags:
- necojackarc
- Codeforces
- Scala
- Ruby
- Python
---

Codeforces への参戦記録です。
今回は C まで無事通りました。

# A. Soldier and Bananas
n ドル持ってて、k \* 1, k \* 2, ..., k \* i と値段が変わっていくバナナを w 個買う時、いくら借金する必要があるかという問題です。

支払い合計を計算し、手持ちの n ドルと比較してやれば答えは出ます。

```scala
object R304_SoldierAndBananas extends App {
  val sc = new java.util.Scanner(System.in)
  val k, n, w = sc.nextInt
  val ans = (1 to w).map(_ * k).sum - n
  println(if (ans > 0) ans else 0)
}
```

# B. Soldier and Badges
n 個の数字が与えられ、それに加算のみを行い全ての数字をバラバラにするには、合計いくつ数字を増やす必要があるかという問題です。

ソートして前から順番に見ていき、常に直前の値より大きくなるようにすれば最短手順でバラバラにできます。

```scala
object R304_SoldierAndBadges extends App {
  val sc = new java.util.Scanner(System.in)
  val n = sc.nextLine().toInt
  val a = sc.nextLine().split(" ").map(_.toInt).sorted

  var ans = 0
  var bef = a(0)
  (1 to a.length - 1).foreach { i =>
    var tmp = 0
    while (bef >= a(i) + tmp)  {
      tmp += 1
      ans += 1
    }
    bef = a(i) + tmp
  }
  println(ans)
}
```

Scala 感のなさすぎるコードを書いてしまった……。

# C. Soldier and Cards
$n (2 \leq n \leq 10)$枚のカード (カードにはそれぞれ1から n までの数値がバラバラに振ってある) を2人のプレイヤーに配る。
お互いのデッキの一番上のカードを比較し、数値の大きい方のプレイヤーがそのターンでの勝者となる。勝者はまず敗者のカードを自分のデッキの一番下に入れる、そのあとその下に自分のカードを入れる。
これを繰り返し、デッキを失った場合そのプレイヤーの負けとなる。

デッキの初期状態が与えられるので、どちらのプレイヤーが何ターン目で勝つのか、もしくは決着がつかないのかを出力する。

## 考え方
入力値が小さいので、バカ正直にシミュレーションをすれば問題なさそう。
「これまでに出現したデッキの状態」が出現したときに引き分けが確定するので、これまでに出現したデッキの状態を覚えておく必要がある。
また、各プレイヤーのデッキが入れ替わった状態も同一の状態とみなすことができる。

## 実装
デッキの状態の記憶は、連想配列 (Hashmap) や 集合 (Set) で行えば良さげ。
また、各プレイヤーのデッキが入れ替わった状態も同一とみなせるということは、順序を持たない集合型でデッキの状態の組を持たせれば良い。

今回は集合型で書くのがめんどくさかったので、単なる配列にデッキの状態を格納し、その順番を入れ替えた配列もこれまで出現したデッキの状態として格納する方針で実装した。

```ruby
require 'set'

def nextState!(winner, loser)
  c1 = winner.shift
  c2 = loser.shift
  winner << c2
  winner << c1
end

n = gets.chomp.to_i
k1 = gets.chomp.split("\s").map(&:to_i).select.with_index { |_, idx| idx != 0 }
k2 = gets.chomp.split("\s").map(&:to_i).select.with_index { |_, idx| idx != 0 }

count = 0
states = Set.new([[k1.dup, k2.dup], [k2.dup, k1.dup]])
while k1.length != 0 && k2.length != 0
  count += 1

  if k1.first > k2.first
    nextState!(k1, k2)
  else
    nextState!(k2, k1)
  end

  if states.include?([k1, k2])
    count = -1
    break
  end

  states << [k1.dup, k2.dup]
  states << [k2.dup, k1.dup]
end

if count == -1
  puts count
else
  puts "#{count} #{k1.length != 0 ? 1 : 2}"
end
```

Scala 感が全く出せないので Ruby で書きました。

- 集合で覚えるより連想配列で覚えた方が出現状態かどうかの確認が早そう
- 状態の組み合わせを文字列化 (ハッシュ化) した方がメモリ消費少なそう

と感じたので、Python で書き直してみました。

```python
def to_s(array):
    return ",".join(map(str, array))

def to_hash(k1, k2):
    return to_s(k1) + ":" + to_s(k2)

def add_states(states, k1, k2):
    states[to_hash(k1, k2)] = True
    states[to_hash(k2, k1)] = True

def nextState(winner, loser):
    c1 = winner.pop(0)
    c2 = loser.pop(0)
    winner.extend([c2, c1])

n = int(raw_input())
k1 = map(lambda x: int(x), raw_input().split(" "))[1:]
k2 = map(lambda x: int(x), raw_input().split(" "))[1:]

count = 0
states = {}
add_states(states, k1, k2)

while len(k1) != 0 and len(k2) != 0:
    count += 1

    if k1[0] > k2[0]:
        nextState(k1, k2)
    else:
        nextState(k2, k1)

    if states.has_key(to_hash(k1, k2)):
        count = -1
        break

    add_states(states, k1, k2)

if count == -1:
    print count
else:
    print u"%d %d" % (count, 1 if len(k1) != 0 else 2)
```

もう少し日本国民に優しい時間に開催してほしい……。

title: "Codeforces #290 Div.1 A"
date: 2015-05-04 14:08:34
tags:
- esplo77
- Codeforces
- Python
---

# A. Fox And Names
解けはしたが、実装力が足りな過ぎた…。

## 問題
[問題](http://codeforces.com/contest/512/problem/A)

n個の名前が順番に与えられる。
名前は辞書順に並んでいるが、使われているアルファベットは通常の順番と異なる。
考えられるアルファベット順の1つを出力せよ。
また、そのような順番が存在しない場合は"Impossible"と出力せよ。

$$1 \leq n \leq 100 \\\
1 \leq |name_i| \leq 100$$

## 考え方
まずサンプル1を考えてみる。

```
3
rivest
shamir
adleman
```

この場合は、[r, s, a]が順番に出てくれば良い。
他のアルファベットの順番はどうでもよいので、rsa+[残り]を出力すれば正解。

次に、サンプル2。

```
10
tourist
petr
（略）
tankengineer
```

このように、同じ文字が離れた位置に来ると、[t, p]が順番に出てきて、[p, t]が順番に出てこなければいけない。
当然矛盾するので、"Impossible"。

基本は掴めてきたので、先頭に同じ文字を含む場合を考える。

```
4
aa
ab
abc
abd
aba
```

この場合、[a, b] [c, d, a]の両方を満たす順番で出てくればよい。
[c,d,a,b]など。

順番の制約を抽出するのは、色々な方法でできそう。
再帰でやるなら、イメージはこんな感じ。
```python
def work(c, l):
    if len(l) <= 1:
        return []

    res = []

    # 重複しない順序 ordered setが使えれば・・・
    order = []
    for name in l:
        if not name[0] in order:
            order.append(name[0])
    res.append(order)

    for head in order:
        res.extend( work(head,
             map(lambda t:t[1:],
                 filter(lambda x:x[0] == head, l))
             )
        )

    return res
```

ordered setが使えないので面倒なことになっているが、
resに制約のリスト（[[a,b,c],[d,e],[f,g]]のような感じ）が返ってくる。


### 制約をマージする
得られた制約は独立しているので、1つの制約にまとめたい。
よく見ると、制約って有向グラフじゃね？となる（制約群は凝視すると隣接リストになる）。
[a,b,c] => (a->b),(b->c)

さて、有向グラフで制約を満たした順番を得るのはよく知られた方法でできる。
そうだね、トポロジカルソートだね。
トポロジカルソートはループを検出するので、[a,b][b,a]などImpossibleな状態も検出できる
（ちなみに、Pythonのトポロジカルソートは、偶然見つけた[kachayev / topological.py](https://gist.github.com/kachayev/5910538)を頂戴しました）。

トポロジカルなアルファベットの順番が得られたら、足りないアルファベットを出力しておしまい。

## まとめ

1. 名前の列から制約を抽出
1. 制約を有向グラフと見なして、トポロジカルソート
1. 制約に含まれない文字を出力

この3ステップで解ける。
nが小さいので、$n^3$で回しても問題なし。

## 実装
[作者の回答](http://ideone.com/KVobNb)を見ると、

- 名前の後ろに" "を付けて処理を簡単にしている
- 制約を得るのにいきなり隣接行列を作っている
- ワーシャルフロイドで経路探索して、2文字間の制約をチェック

など、色々工夫されている。
自分のコードを載せるまでもないので、こちらを参照ください。

## 結果
Python2: 77ms, 1200KB
title: "Codeforces #292 Div.1 A"
date: 2015-05-04 20:46:40
tags:
- esplo77
- Codeforces
- Python
---

# A. Drazil and Factorial

ごり押しで通した。久しぶりに一発で通ったかも。

## 問題
$F(x)$は各桁の階乗を掛け合わせた値を返す。
値aが与えられたとき、$F(a)=F(x)$となるxの中で最大の値を求めよ。
なお、xは0,1を含んではいけない。

$$1 \leq n \leq 15$$

## 考え方
とっかかりを見つけるため、サンプル1を書き出してみる。
$$
1! \cdot 2! \cdot 3! \cdot 4! \\\
= 1 \cdot 2 \cdot 1 \cdot 3 \cdot 2 \cdot 1 \cdot 4 \cdot 3 \cdot 2 \cdot 1 \\\
= 4 \cdot (3 \cdot 3) \cdot (2 \cdot 2 \cdot 2) \\\
= (3 \cdot 3) \cdot (2 \cdot 2 \cdot 2 \cdot 2 \cdot 2) \\\
= (3 \cdot 2) \cdot (3 \cdot 2) \cdot 2 \cdot 2 \cdot 2
$$

なるほど。階乗をばらして再構成すればよさそうだ。

出力する値を最大化するためには、桁の値を大きくするより、桁数を増やした方がよい。
そのため、なるべく素因数分解すべき。

素因数分解したら、大きな素数から順番に階乗を作ってゆく（5があれば5!から作る）。
この方法でできないケースは、$18 = 3 \cdot 3 \cdot 2$など、大きい数の方が多く生まれてしまう場合。
運よく0～9にそんなケースはないので、特に気にしなくて良い。

素因数分解を書くのが大変そうだが、0～9しか取りうる値がないため、全ケース書き出した。
O(n)程度の処理（適当）なので、時間的にも問題なし。

## コード

```python
# -*- coding: utf-8 -*-
import sys
import heapq
from collections import defaultdict
from collections import OrderedDict
import copy
from collections import deque


class Solver(object):

    def run(self):
        n = read(int)
        a = map(int, list(read(str)))

        def addElem(elem, i, isAdd):

            func = lambda i: i
            if not isAdd:
                func = lambda i: -i

            if i in [0, 1]:
                pass
            elif i == 2:
                elem[2] += func(1)
            elif i == 3:
                elem[3] += func(1)
            elif i == 4:
                elem[2] += func(2)
            elif i == 5:
                elem[5] += func(1)
            elif i == 6:
                elem[2] += func(1)
                elem[3] += func(1)
            elif i == 7:
                elem[7] += func(1)
            elif i == 8:
                elem[2] += func(3)
            elif i == 9:
                elem[3] += func(2)

        # 0..loop
        def addElemLoop(elem, isAdd, loop):
            for t in xrange(loop+1):
                addElem(elem, t, isAdd)
            return elem

        elems = defaultdict(int)
        for i in a:
            addElemLoop(elems, True, i)

        res = []
        keys = sorted(elems.keys(), reverse=True)
        for k in keys:
            for i in xrange(elems[k]):
                res.append(k)
                addElemLoop(elems, False, k)

        print("".join(map(str,res)))`

####################################
# ここからは見ちゃダメ！

def read(foo):
    return foo(raw_input())
def readarray(foo):
    return [foo(x) for x in raw_input().split()]
def dbg(a):
    sys.stderr.write(str(a))
def dbgln(a):
    sys.stderr.write(str(a) + "\n")

if __name__ == '__main__':
    Solver().run()
```

## 結果
77ms, 944KB

[解説](http://codeforces.com/blog/entry/16468)見ると、より効率よく0～9の分解を行っている。なるほど。


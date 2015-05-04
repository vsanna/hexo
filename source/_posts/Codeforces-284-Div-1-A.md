title: "Codeforces #284 (Div. 1) A"
date: 2015-05-04 16:36:06
tags:
- esplo77
- Codeforces
- Python
---

# A. Crazy Town

簡単な幾何。ゲームでもよくある処理ですね。

## 問題

$a_ix + b_iy + c_i = 0 $（ただし、a,bの両方が0にはならない）というn個の直線でブロックが区切られている。
ブロックを横断するのに1ステップ使う。
直線の交点を通る際は、直線の数だけステップを使う。
座標(x1,y1）に家、座標(x2,y2）に大学があるとき、家から大学に行くときの最短ステップはいくらか。
なお、家と大学は（ブロックを分割している）直線上にはない。

$$
1 \leq n \leq 300 \\\
-10^6 \leq x1, y1, x2, y2, a_i, b_i, c_i \leq 10^6
$$


## 考え方
図を描いてみると分かり易いが、家と大学をまっすぐ繋ぐのが最短ステップになる。
交点がややこしそうに見えるが、特殊な処理はないため、それぞれ独立した直線と考えればよい。

ステップ数 -> 直線と交わった回数
なので、家と大学を繋いだ線分と、各直線が交わるかどうかを判定するだけでオーケー。

### 交差判定
[ここを参照](http://www5d.biglobe.ne.jp/~tomoya03/shtml/algorithm/Intersection.htm)
交差判定は点同士の処理になるが、問題は直線の方程式で与えられている。
そのため、直線から適当な2点をピックアップする必要がある。
a,bのいずれかは0でないというありがたーい制約があるので、以下のようなコードで直線を2点に変換する。

```python
def line2point(a, b, c):
    c = float(c)
    def _workX(x):
        # a*x + b*y + c = 0
        return (-1*c - a*x) / b
    def _workY(y):
        # a*x + b*y + c = 0
        return (-1*c - b*y) / a

    if b != 0:
        return _workX
    else:
        return _workY
```
このようなカリー化した関数を用意し、
```python
func = line2point(a, b, c)
if b != 0:
    points.append( (0, func(0)) )
    points.append( (1, func(1)) )
else:
    points.append( (func(0), 0) )
    points.append( (func(1), 1) )
```
適当に0,1を与えた2点を作る。

なお、**もっと簡単に解けます**。
最後に記載。

n個の直線との交差判定はo(n)なので余裕。
C++はlong longで取らないと座標値で死に得るので注意。

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
        p1 = tuple(readarray(int))
        p2 = tuple(readarray(int))
        n = read(int)
        lines = [readarray(int) for i in xrange(n)]

        def line2point(a, b, c):
            c = float(c)
            def _workX(x):
                # a*x + b*y + c = 0
                return (-1*c - a*x) / b
            def _workY(y):
                # a*x + b*y + c = 0
                return (-1*c - b*y) / a

            if b != 0:
                return _workX
            else:
                return _workY

        res = 0
        for line in lines:
            points = []
            # convert line into points
            a, b, c = line
            func = line2point(a, b, c)
            if b != 0:
                points.append( (0, func(0)) )
                points.append( (1, func(1)) )
            else:
                points.append( (func(0), 0) )
                points.append( (func(1), 1) )

            if lineIntersect(points[0], points[1], p1, p2):
                res += 1

        print(res)

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
# l1 to l2 -> line
# ls1 to ls2 -> line segment
def lineIntersect( l1, l2, ls1, ls2 ):
    l1x, l1y = l1
    l2x, l2y = l2
    ls1x, ls1y = ls1
    ls2x, ls2y = ls2
    if (((l1x - l2x) * (ls1y - l1y) + (l1y - l2y) * (l1x - ls1x)) *
        ((l1x - l2x) * (ls2y - l1y) + (l1y - l2y) * (l1x - ls2x)) > 0):
        return False
    return True
# p1 to p2 -> line segment
# p3 to p4 -> line segemnt
def intersect( p1, p2, p3, p4 ):
    if lineIntersect(p1, p2, p3, p4) and lineIntersect(p3, p4, p1, p2):
        return True
    return False

if __name__ == '__main__':
    Solver().run()
```

## 結果
77ms, 944KB

## もっと簡単に交差判定
直線の方程式に座標を代入すると、その直線に対して左右どちらにあるかが正負で返ってくる。
というわけで、わざわざ直線と線分の交差判定関数を用意するのはオーバーキル。
家と大学の座標を方程式に与えて、それぞれ正負が逆なら交差する、というだけで十分。

時間がもったいなかったけど、カリー化が活用できたからいいかー（現実逃避
title: "Codeforces #301 (Div. 2 Only) D"
date: 2015-05-05 01:08:11
tags:
- esplo77
- Codeforces
- Python
---

# D. Bad Luck Island

確率は苦手。

## 問題
[問題](http://codeforces.com/contest/540/problem/D)
r人のグー族、s人のチョキ族、p人のパー族がいる。
異なる種族と出会うと、じゃんけんで負ける方が死ぬ。
最終的にその種族のみが生き残る確率を求めよ。
なお、異なる種族とは等確率で出会う。

$1 \leq r, s, p \leq 100$

## 考え方

ゼミで見たことあるような確率DP。
制約的に、$100^3 = 10^6$でおそらく実装するのだろう。

### DP

まず、dp[i][j][k]を「グーがi人、チョキがj人、パーがk人という状況になる確率」と定義する。

遷移の例としてグーとチョキが出会う確率を考える。
全出会いの数は、$i \cdot j + j \cdot k + k \cdot i$で表される。
その中でグーとチョキが出会うのは、$i \cdot j$通り。
会うとチョキが死に1人減るので、以下の漸化式で表すことができる。

$$dp[i][j-1][k] = \\frac{i \cdot j}{i \cdot j + j \cdot k + k \cdot i} \cdot dp[i][j][k]$$

これを3種類の出会いそれぞれに対して行えばよい。
（なお、p[i][j][k] = ...の形にまとめられはするが、TLEを食らった。本記事末に記載）

$O(n^3)$なので、おそらく間に合う。
C++なら余裕なはず。

### 集計
DPテーブルが埋まったら、ある種族だけ1人以上いて、他の種族が0な状況の合計値を取れば、それぞれが答えになる。
$O(n)$なので気にしない。

### 出力
小数点以下9桁までは出さないといけない。
サンプルでは12桁まで出しているので、それに倣う。
```python
"%3.12f" % num
```
とすると、小数点以上3桁、小数点以下12桁で出してくれる。
なお、Pythonのfloatは53bitの精度を持っているそうなので、doubleかどうかを気にしなくていい。

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
        r, s, p = readarray(int)

        dp = [[[0.0]*101 for row in xrange(101)] for col in xrange(101)]
        dp[r][s][p] = 1.0

        for i in sorted(xrange(101), reverse=True):
            for j in sorted(xrange(101), reverse=True):
                for k in sorted(xrange(101), reverse=True):

                    TOTAL = i*j + j*k + k*i
                    if dp[i][j][k] != 0.0 and TOTAL != 0:
                        # r kills s
                        dp[i][j-1][k] += float(i*j)/TOTAL * dp[i][j][k]

                        # s kills p
                        dp[i][j][k-1] += float(j*k)/TOTAL * dp[i][j][k]

                        # p kills r
                        dp[i-1][j][k] += float(k*i)/TOTAL * dp[i][j][k]

        rp = 0.0
        sp = 0.0
        pp = 0.0

        for i in xrange(101):
            if dp[i][0][0] != 0.0:
                rp += dp[i][0][0]
        for i in xrange(101):
            if dp[0][i][0] != 0.0:
                sp += dp[0][i][0]
        for i in xrange(101):
            if dp[0][0][i] != 0.0:
                pp += dp[0][0][i]

        print("%2.12f %2.12f %2.12f" % (rp, sp, pp))


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
1855ms, 22300KB
危ない。$10^6$でこれなので、なかなか厳しい。

ちなみに、dpをきれいにしようとして、中で色々やるとTLEする。

## TLEしたやつ（DP部分のみ)
```python
def getTotal(i, j, k):
    return i*j + j*k + k*i
def getPercent(func):
    def _getPercent(i, j, k):
        total = getTotal(i, j, k)
        if total == 0:
            return 0
        return float(func(i, j, k)) / total * dp[i][j][k]
    return _getPercent

for i in sorted(xrange(100), reverse=True):
    for j in sorted(xrange(100), reverse=True):
        for k in sorted(xrange(100), reverse=True):

            dp[i][j][k] += \
                getPercent(lambda i,j,k:i*j)(i, j+1, k) \
                + getPercent(lambda i,j,k:j*k)(i, j, k+1) \
                + getPercent(lambda i,j,k:k*i)(i+1, j, k)
```
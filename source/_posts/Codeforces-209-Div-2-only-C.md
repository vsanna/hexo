title: "Codeforces #209 Div.2 (only) C"
date: 2015-05-01 22:05:48
tags:
- esplo77
- Codeforces
- Python
---

何故かお気に入りに入ってた問題。当時C++で解いたので、Pythonで解き直してみた。

# C. Prime Number
## 問題概要

$$ \\frac{1}{x^{a_1} } + \\frac{1}{x^{a_2} } + ... + \\frac{1}{x^{a_n} } = \\frac{s}{t} \\\
(t = x^{a_1+a_2+...+a_n}) $$
とするとき、sとtの最大公約数を$10^9+7$で割った余りを求めよ。

$$ 1 \leq n \leq 10^5 \\\
2 \leq x \leq 10^9 \\\
0 \leq a_1 \leq a_2 \leq ... \leq a_n \leq 10^9$$

## 考え方

イマイチどうしたらいいか分からないので、とりあえずn=3、x=3、a=[1,2,3]の時を書き出してみます。

$$ \\frac{1}{3^1} + \\frac{1}{3^2} + \\frac{1}{3^3} \\\
= \\frac{3^2\cdot3^3 + 3^3\cdot3^1 + 3^1\cdot3^2}{3^1\cdot3^2\cdot3^3} \\\
= \\frac{3^3(3^2 + 3^1 + 1)}{3^6}
$$

この場合、答えは$3^3$ですね。
どうやら、分子の指数の最小値が答えになりそうです。

さて、上の例で言う$(3^2 + 3^1 + 1)$が影響を与える場合を考えてみましょう。
考えてみると分かりますが、カッコ内が$(1 + 1 + 1)$などであれば、答えが変わってしまいますね。

というわけで、以下の条件を満たせば良さそうです。
1. 基本は最小の指数の値を求めれば良い
2. 最小の指数の値が$x\times n$個ある場合は、$x+1$を$x / n$個に変換

## 注意点
### 実装上の注意
[1, 1, 1, 1, 3] => [2, 2, 3] => [3, 3] => [4]
というような変換をしないといけないですが、配列でやるとかなり複雑な処理になってしまいいます。
priority queueを使うとこういう処理は比較的楽に書けます。
Pythonでは微妙に扱いにくいインターフェースのheapqがそれです。
priority queueは競プロ頻出なので知っておくと何かと便利です。

### 累乗
この制約だと、何も考えないと最悪時に$(10^9)^(10^18)$というえげつない計算が出てきます。
累乗の計算がO(n)だと即死ですね。
O(1)で累乗を行うおなじみの関数があるので、C++などで書く際はこれを使わないといけません。

```c++
// n^i
int fpow(ll n, ll i) {
	ll res = 1ll;
	ll x = n;
	while (i != 0ll) {
		if ((i & 1) == 1ll)
			res = (res * x) % M;
		x = (x * x) % M;
		i >>= 1;
	}
	return (res % M);
}
```
なお、Pythonでは、組み込みのpow()の第三引数にMを与えると同じことをやってくれます。神。

### 他の罠
分母より大きな数は公約数になりません。
忘れがちなので注意。

## コード
```python
# -*- coding: utf-8 -*-
import math
import sys
from collections import defaultdict
import collections
import heapq


class Solver(object):

    def run(self):
        n, x = readarray(int)
        a = readarray(int)

        suma = sum(a)
        degree = map(lambda x: suma-x, a)
        heapq.heapify(degree)

        prev = heapq.heappop(degree)
        count = 1
        while degree:

            head = heapq.heappop(degree)

            if prev != head:
                if count % x == 0:
                    for i in xrange(count/x):
                        heapq.heappush(degree, prev+1)
                    count = 1
                else:
                    break
            else:
                count += 1

            if count % x == 0:
                heapq.heappush(degree, head + 1)
                count = 0

            prev = head

        print(pow(x, min(suma, prev), int(1e9+7)))


################################################################################
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

## 実行結果

Python: 967ms, 11200KB
めちゃくちゃギリギリです。
運が悪いとTLE食らうレベル。
無駄な処理が入ってるのかな…。

ちなみに、全く同じコードをC++0xで書くと、
C++0x: 31ms, 2500KB

C++使いたくなってきた。
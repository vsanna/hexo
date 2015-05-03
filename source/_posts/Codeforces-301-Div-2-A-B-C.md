title: "Codeforces #301 Div.2 A, B, C"
date: 2015-05-03 20:21:41
tags:
- esplo77
- Codeforces
- Python
---

# A. Combination Lock
## 問題概要

n桁ダイヤル式の鍵がある。
初期状態から何回で開錠できるか求めよ。

$1 \leq n \leq 100$

## 考え方
いわゆる「書くだけ」問題。
9 -> 1 に変えたい時に注意。
min( 9-1, 10-(9-1) )で最小の回数は求まる。

## コード
```python
# -*- coding: utf-8 -*-
import sys
# 必要なimportを増やす


class Solver(object):

    def run(self):
        n = read(int)
        start = read(str)
        end = read(str)

        def count((a, b)):
            t = abs(int(a) - int(b))
            return min(t, 10-t)

        print(
            sum(map(count, zip(list(start), list(end))))
        )


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
61ms, 4500KB

# B. School Marks

## 問題

n回のテストを受けて、点数の合計がx以下、中央値がy以上になるようにせよ。
なお、k回は既に受けている。

$$ 1 \leq n \leq 999 (n = 2m+1) \\\
0 \leq k \leq n \\\
1 \leq p \leq 1000
$$

## 考え方
「中央値がy以上」 = 「n個の数列中、y以上の値が半分以上ある」
と言い換えができる。
合計値はなるべく小さい方がうれしいので、

- y以下の値を増やしたい -> 1
- y以上の値を増やしたい -> y

をそれぞれ加えればよい。
O(n)。

## コード
```python
# -*- coding: utf-8 -*-
import sys
# 必要なimportを増やす


class Solver(object):

    def run(self):
        n, k, p, x, y = readarray(int)
        a = readarray(int)

        suma = sum(a)
        lowerCount = len(filter(lambda x:x<y, a))
        upperCount = len(a) - lowerCount

        res = []
        for i in xrange(n-k):
            if lowerCount > upperCount \
                    or (lowerCount == upperCount and i == (n-k-1)):
                res.append(y)
                upperCount += 1
            else:
                res.append(1)
                lowerCount += 1

        total = suma + sum(res)
        if total > x or lowerCount > upperCount:
            print(-1)
        else:
            print(" ".join(map(str, res)))


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
62ms, 4500KB
正答者少なかったけど、比較的簡単な気がする。

# C. Ice Cave
読解ミスってめちゃ悩んだ。

## 問題
$n \times m$のマップに、「もろい床(X)」「普通の床(.)」の2種類がある。
「普通の床」は一度通ると「もろい床」になり、「もろい床」に移動すると下に落下してしまう。
(r1, c1)から移動を開始し（開始地点は既にもろい床になっている）、(r2, c2)の座標で下に落ちたい。
可能なら"YES"、無理なら"NO"と出力せよ。

$1 \leq n,m \leq 500$

## 考え方
場合分けゲー。

まず何も考えず、Xを通らないでスタートからゴールまで行ける道があるかを考える。
道が無い場合は落下不可能。
道がある場合は、ゴールとその周辺の状況が問題となる。

- ゴールがX -> 落下可能
- ゴールが. -> 周囲に.が2マス以上あれば落下可能

なぜ2マスかというと、ゴールに到達するために通る床、ゴールをxにしてから再度踏むために経由する床の2つが必要なため。

基本はこれで良いが、以下の2パターンも考慮する必要がある。

- スタートとゴールが同じ位置 -> 周囲に1マスあれば落下可能
- スタートとゴールが隣接
	- ゴールがX -> 落下可能
	- ゴールが. -> 周囲に1マスあれば落下可能（スタート地点はXになっているため）

ちなみに、2個目のパターンは、スタート地点を.に置き換えておくことで通常時と同じパターンにまとめられる。
お好みでどうぞ。

## コード
```python
# -*- coding: utf-8 -*-
import sys
import heapq


class Solver(object):

    dx = [1, 0, -1, 0]
    dy = [0, 1, 0, -1]
    dir = zip(dx, dy)

    def run(self):
        n, m = readarray(int)
        cave = []
        for i in xrange(n):
            cave.append(list(read(str)))
        start = tuple(map(lambda x:x-1, readarray(int)))
        goal = tuple(map(lambda x:x-1, readarray(int)))
        cave[start[0]][start[1]] = "."

        def makeAdjacent((r, c)):
            for (dy, dx) in self.dir:
                if 0 <= r + dy < n and 0 <= c + dx < m:
                    yield (r+dy, c+dx)

        def countAround((r, c)):
            return map(
                lambda (ra,ca):cave[ra][ca],
                makeAdjacent((r,c))
            ).count(".")

        def isPath((rs,cs), (rg,cg)):
            d = [[None] * m for i in xrange(n)]

            que = []
            que.append((rs, cs))

            while que:
                r, c = que.pop(0)
                if (r, c) == (rg, cg):
                    return True

                for t in makeAdjacent((r,c)):
                    ra, ca = t
                    if cave[ra][ca] == "." or t == goal:
                        if d[ra][ca]:
                            continue
                        d[ra][ca] = True
                        que.append(t)
            else:
                return False


        def solve():
            if start == goal:
                if countAround(start) > 0:
                    return True
                else:
                    return False
            elif isPath(start, goal):
                if cave[goal[0]][goal[1]] == "X":
                    return True
                elif countAround(goal) > 1:
                    return True
                else:
                    return False
            else:
                return False

        t = "YES" if solve() else "NO"
        print(t)


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
904ms, 6784KB

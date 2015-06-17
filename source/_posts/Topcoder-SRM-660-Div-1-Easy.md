title: "Topcoder SRM 660 Div.1 Easy"
date: 2015-06-16 00:27:49
tags:
- esplo77
- Codeforces
- C++11
---

# 250 Coversta

制約がなんとも言えず、色々な解き方ができる模様。
本番では解けなかったけど。
というかx, yとかいう紛らわしい引数名はやめていただきたい……。

## 問題

[問題文](http://community.topcoder.com/stat?c=problem_statement&pm=13807)

四角のフィールドがある。
フィールドはn行m列のセルで出来ており、各セルには0～9の値が決められている。
また、x,yという配列（長さは共にk）が与えられる。
ポイント(i, j)は$(i + x[k], j + y[k])$ （kは有効なインデックス）のセルを網羅することができる。

2つのポイントを選ぶ時、網羅したセルの合計値の最大を求めよ。
なお、2つのポイントで同じセルを網羅した場合でも、値は1度しかカウントされない。

$$
2 \leq n, m \leq 100 \\\
1 \leq k \leq 10
$$


## 考え方
まず最初に思いつくのは、ポイントを2つ取り出し、重なりを考慮して値を求め、その最大値を求める方法。
これは、ポイントを2つ取り出すので$O(n*m*n*m)$、重なりを考慮して値を求めるので$O(k*k)$なので、ループ回数は$10^10$になり間に合わない。

そこで、以下の特徴を利用する。
あるポイントを1つ選ぶと、k個のセルを網羅する。
2つのポイントA, Bを選ぶと、重なりのパターンは最大で$k^2$個ある。
なぜなら、同じセル同士が重なる場合は1通りしか無いため。
例えば、(Ar + x[0], Ac + y[0]) と (Br + x[3], Bc + y[3])が重なる場合、A, Bの座標は一点に定まる。
そのため、(Ar + x[0], Ac + y[0])とBの各セル、(Ar + x[1], Ac + y[1])とBの各セル、...というパターンで重なる場合は全て網羅できる。

さて、重なりのパターンが$k^2$ということは、$k^2+1$パターンを取り出せば、必ず1つは重ならないパターンが存在する。
これを利用すると、以下の方法で最大の2ポイントが求まる。

1. 各セルで、網羅するマスの合計値を求めておく $O(n \cdot m \cdot k)$
2. 合計値が降順になるようソートする $O(n \cdot m \cdot \log(n \cdot m))$
3. 先頭から$k^2 + 1$個を取り出し、それぞれの組み合わせで重なっている部分の値を求める $O(k^2 \cdot k \codt k)$
4. 3で求めた2ペアの合計値から重なっている値を除く
5. それらの最大値が答え

コードに落とす時は、セル数が$k^2+1$より少ない場合こ考慮したり、値と座標のペアの配列を作らないといけなかったりと、意外と実装が重たいので注意。

## コード

```C++
class Coversta {
public:

    int place(vector<string> a, vector<int> x, vector<int> y) {
        int w = a[0].size();
        int h = a.size();

        vector<pair<int, pii> > sums;
        rep(i, h) {
            rep(j, w) {

                int s = 0;
                rep(k, x.size()) {
                    int tr = i + x[k];
                    int tc = j + y[k];

                    if (0 <= tr && tr < h && 0 <= tc && tc < w) {
                        s += a[tr][tc] - '0';
                    }
                }

                sums.push_back(mp(s, mp(i, j)));
            }
        }

        sort(all(sums), greater<pair<int, pii> >());

        int k = x.size();
        int mx = min(k * k + 1, w * h);
        int res = 0;
        rep(i, mx) {
            REP(j, i + 1, mx) {

                // search duplicating area
                pii p1 = sums[i].second;
                pii p2 = sums[j].second;

                int minus = 0;
                rep(k1, x.size()) {
                    rep(k2, x.size()) {
                        int tr1 = p1.first + x[k1];
                        int tc1 = p1.second + y[k1];

                        int tr2 = p2.first + x[k2];
                        int tc2 = p2.second + y[k2];

                        if(tr1 == tr2 && tc1 == tc2) {
                            if (0 <= tr1 && tr1 < h && 0 <= tc1 && tc1 < w) {
                                minus += a[tr1][tc1] - '0';
                            }
                        }
                    }
                }

                res = max(res, sums[i].first + sums[j].first - minus);
            }
        }

        return res;
    }
};
```

## 感想
これも本番で解けなかったし、Div.1問題が解ける気がしない。

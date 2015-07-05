title: "Codeforces #308 Div.2 D"
date: 2015-07-05 18:04:23
tags:
- esplo77
- Codeforces
- C++11
---

# D. Vanya and Triangles


## 問題
平面に$n$個の点($x_i$, $y_i$)がある。
それらの点を互いにつないだ時、いくつの三角形ができるか求めよ。

$$
1 \leq n \leq 2000 \\\
-100 \leq x_i, y_i \leq 100
$$


## 考え方

## 方法1
三角形ができるかを全ての点の組み合わせで試せば良い。
$n$から3点を取り出し、一直線上に並ばないなら三角形になる。
nC3なので、$O(n^3)$。

$n = 2000$の時は間に合わない……。
と思ったら、editorialによるとC++で間に合ってしまうらしい。
4秒制限は意外と長い。
これじゃD問題にならないので、次の方法で解く。

## 方法2
基本的には「全通り試して、ダメな組み合わせを取り除く」という、方法1と同じ手順を行う。
ただし、オーダーを下げるため工夫をする。

1. 1点を決める
1. 全ての残りの点との組み合わせで三角形ができると仮定し、答えに加算
1. 同一直線上にある点をカウントし、答えから減算

手順2では、$n-1$個の点から2点を選ぶので、$\frac{n \cdot (n-1)}{2}$を加算すれば良い。
手順3では、同一直線上にあるかどうかを、手順1で決めた点からの角度で判別すると点を2つ取り出さなくてもよくなる。
ある点から見た角度が同じ（0°と180°は同一直線上なので同じとみなす）点がm個あるとすると、m個から2個取り出した場合を除外すればよい。

同じ角度かどうかは、ベクトルで判別すると良い。
dx, dyをgcdで割って正規化（？）した後、$dx < 0$ならdx,dyを正負反転すると、同じ直線上にある点であれば同じdx,dyになる。

この方法では同じ三角形を3回（それぞれの点で）作ってしまうので、最後に3で割ってあげる。

ループ回数は、高々$n^2 \cdot \log 200$なのでゆったり間に合う。

## コード
```C++
int gcd(int a, int b) {
    if(b == 0)
        return a;
    return gcd(b, a % b);
}


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    cin >> n;

    int x[n], y[n];
    rep(i,n)
        cin >> x[i] >> y[i];

    ll res = 0;
    rep(i,n) {

        int size = n - 1;
        res += ((ll)size * (size-1)) / 2;

        map< pii, int > rads;
        rep(j, n) {
            if( i == j)
                continue;
            int dx = x[j] - x[i];
            int dy = y[j] - y[i];

            if(dx == 0)
                dy = 1;
            else if(dy == 0)
                dx = 1;
            else {
                int g = gcd(abs(dx), abs(dy));
                dx /= g;
                dy /= g;

                if(dx < 0) {
                    dx *= -1;
                    dy *= -1;
                }
            }

            rads[mp(dx, dy)]++;
        }

        for( auto t : rads )
            res -= ((ll)t.second * (t.second-1)) / 2;
    }

    cout << res / 3  << endl;

    return 0;
}
```
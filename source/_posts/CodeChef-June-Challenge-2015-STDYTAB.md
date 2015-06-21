title: "CodeChef June Challenge 2015 STDYTAB"
date: 2015-07-02 23:07:04
tags:
- esplo77
- CodeChef
- C++11
---

# Steady tables

## 問題

[問題文](http://www.codechef.com/problems/STDYTAB)

N行M列の表がある。
各行は1～Nで、各列は1～Mで表される。
ある表がsteadyであるとは、以下の条件を満たす場合である。

- i行目の要素の合計 >= (i-1)行目の要素の合計
- N行目の要素の合計がM以下

N、Mが与えられるとき、steadyな表のパターン数を$10^9$で割った余りを出力せよ。

## 考え方

以下の2つの問題に分けて考えてみる。

1. 各行の合計のパターンを列挙
2. 各行の合計が与えられた時、組み合わせのパターン数を算出

1は、(i-1)行目の合計はi行目の合計より小さいため、N行目の合計を定め、再帰で辿ると楽に全パターンを辿る事ができる。

2は、合計xをM個のどこかに重複を許して入れるため、$(x+1) H(m-1)$で求められる。
変形しておいて、$(x+m-1) C (m-1)$となる。

というわけで、iが大きい方から再帰をスタートし、各合計のパターンを巡る中で組み合わせのパターン数を掛けていけばよい。
以下は再帰部分のサンプル。

```C++
int work(int i, int x) {
    if(i < 0)
        return 1;
    if(x < 0)
        return 0;

    if(memo[i][x] != -1) {
        return memo[i][x];
    }

    return memo[i][x] =
            ((((ll)nCr[x+m-1][m-1] * work(i-1, x)) % MOD) + work(i, x-1)) % MOD;
}
```

ただ、これで投げてみると、1つだけサンプルがTLEするので、DPで書きなおさないといけない模様。
再帰はトップダウンなので、ボトムアップに変形する。

### nCr

modを取るnCrは定義通りに求める方法がよく使われる（気がする）。
階乗を収めた配列を用意し、modの逆元を使って計算をする。
ただ、これだとmodが素数でないと計算が（多分）大変。

そこで、パスカルの三角形で示されている式を利用する。
[Wikipedia - パスカルの三角形](https://ja.wikipedia.org/wiki/%E3%83%91%E3%82%B9%E3%82%AB%E3%83%AB%E3%81%AE%E4%B8%89%E8%A7%92%E5%BD%A2)

$nCr = (n-1)Cr + (n-1)C(r-1)$
これを記録しながら計算すると、nCrの2次元配列が出来上がる。
今回は制約的に、こっちの方法で計算しておくとよい。

## コード

```C++
const int MOD = 1000000000;
const int MAX = 4003;
const int DPMAX = 2001;
int nCr[MAX][MAX];
int dp[DPMAX][DPMAX];
 
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);
 
    memset(nCr, 0, sizeof(nCr));
    rep(i, MAX) nCr[i][0] = 1;
    REP(i, 1, MAX)
        REP(j, 1, MAX)
            nCr[i][j] = ((ll)nCr[i-1][j] + nCr[i-1][j-1]) % MOD;
 
    int t;
    cin >> t;
    rep(__t, t) {
        int n=0, m=0;
        cin >> n >> m;
 
        memset(dp, -1, sizeof(dp));
 
        rep(i, DPMAX)
            dp[0][i] = nCr[i+m-1][m-1];
        REP(i, 1, n) {
            int s = 0;
            rep(j, m+1) {
                s = ((ll)s + dp[i-1][j]) % MOD;
                dp[i][j] = ((ll)nCr[j+m-1][m-1] * s) % MOD;
            }
        }
 
        int res = 0;
        rep(i, m+1) {
            res = (res + dp[n-1][i]) % MOD;
        }
 
        cout << res << endl;
    }
 
    return 0;
} 
```
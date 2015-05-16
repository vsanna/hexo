title: "Codeforces #295 (Div. 1) A"
date: 2015-05-16 16:20:52
tags:
- esplo77
- Codeforces
- C++11
---

# A. DNA Alignment

バイオッ

## 問題

<http://codeforces.com/contest/521/problem/A>

ACTGの4文字で構成される、長さnの文字列sがある。
同じ長さの文字列tを用意し、文字をシフトさせたものと比較して一致した文字数をカウントする。
例えば、s=AGC、t=CGTであれば、
sを循環させた"AGC", "GCA", "CAG"、
tを循環させた"CGT", "GTC", "TCG"
のそれぞれを比較（9通り）し、一致する文字数（この例では6）を求める。

この求めた数値が最大になるtはいくつ存在するか。
答えは$10^9+7$での剰余を取った値を出力せよ。

$$
1 \leq n \leq 10^5
$$

## 考え方

よくわからないので、色々パターンを考えてみる。
例えば、最大になりそうなs=AGG、t=AGGを考える。

||AGG|GGA|GAG|
|:-:|:-:|:-:|:-:|
|AGG|3|1|1|
|GGA|1|3|1|
|GAG|1|1|3|

なんとなく、1が勿体無い。
s=AGG、t=GGGではどうか。

||AGG|GGA|GAG|
|:-:|:-:|:-:|:-:|
|GGG|3|2|2|
|GGG|2|3|2|
|GGG|2|2|3|

最大っぽい。

改めて考えてみると、「循環させて9通りチェックする」=「$t_i$と$s_0, s_1, s_2$を比較することを3回繰り返す」ことになる。
そのため、sに最も多く含まれている文字を使うと最大になることがわかる。

sに最も多く含まれている文字を全通り並べると答えになるので、累乗計算を高速化すれば$O(n)$で計算できる。

## コード

```C++
// a^k
ll fpow(ll a, ll k, int M) {
    ll res = 1ll;
    ll x = a;
    while (k != 0) {
        if ((k & 1) == 1)
            res = (res * x) % M;
        x = (x * x) % M;
        k >>= 1;
    }
    return res;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    string str;
    cin >> n >> str;

    int cnt[4] = {0};
    rep(i, n)
        ++cnt[ string("ACGT").find(str[i]) ];

    int mx = *max_element(cnt, cnt+4);
    int mxchars = count(cnt, cnt+4, mx);

    cout << fpow(mxchars, n, int(1e9)+7) << endl;

    return 0;
}
```

## 結果
31 ms, 300 KB
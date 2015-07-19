title: "Codeforces #312 C"
date: 2015-07-19 01:31:03
tags:
- esplo77
- Codeforces
- C++11
---

# C. Amr and Chemistry

ﾄｹﾅｶｯﾀ……。
良いところまでは行っていたが、考えが一歩届かなかった感じ。
良問な感じする。

## 問題

$n$個の要素を持つ数列$a$がある。
要素を2倍、またはfloor($\frac{a_i}{2}$)する処理をし、全ての要素を同じ値にするとき、最小の処理回数を答えよ。

$$
1 \leq n \leq 10^5 \\\
1 \leq a_i \leq 10^5
$$

## 考え方

2倍、または1/2する……。
これは……。bitだ！
と気づけるかどうかが肝。

全要素が同じ値になったとき、その値はビット表記で
（全要素のprefix）* （0がいくつか）
となる。
なぜなら、101という値があったとき、
101, 10, 1, 100, 1000, 1000, ....
という値にしか変化させられないためである。

また、結果の値は配列$a$の最大値を超えることは無い。
例えば、a=[101, 1000]があったとき、計算中に1010にする必要は無く、101->10->100->1000とするため、1000を超えない。
仮に両方10000にしたとしても、1000を計算過程で通っているはずなので、明らかに無駄がある。

以上より、以下の流れで解けば良いことがわかる。

1. 全要素のprefixを求める
2. prefixの最大ビット数を求める（=c）
3. 一緒になった値を仮定する（prefix * 2^j = target）
4. $a_i$ -> targetにする処理回数を求める

なお、$a_i$からtargetにする処理は、
「$a_i$とtargetのprefixを求める」「$a_i$ -> prefix の処理回数を求める」「prefix -> targetの処理回数を求める」という3ステップに分けると処理しやすい。

## コード

```C++
int pref(int a, int b){
    if(a == b)
        return a;
    if(a > b)
        a /= 2;
    else
        b /= 2;
    return pref(a, b);
}

int conv(int a, int b) {
    int count = 0;

    int p = pref(a,b);
    // a -> p
    while(a > p)
        count++, a/=2;

    // p -> b
    while(b > p)
        count++, p*=2;

    return count;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    cin >> n;

    int a[n];
    rep(i,n) {
        cin >> a[i];
    }

    int p = a[0];
    REP(i, 1, n) {
        p = pref(p, a[i]);
    }

    int bitlen = 1;
    int tmpp = p;
    while(tmpp > 1) {
        bitlen++;
        tmpp /= 2;
    }

    int res = INT_MAX;
    rep(i, 31-bitlen) {
        int target = p * fpow(2, i);
        int count = 0;
        rep(j, n) {
            count += conv(a[j], target);
        }
        res = min(res, count);
    }

    cout << res << endl;

    return 0;
}
```

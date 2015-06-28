title: "Topcoder SRM 661 Div.1 Easy"
date: 2015-06-25 23:09:15
tags:
- esplo77
- Topcoder
- C++11
---

# 250pt MissingLCM

簡単ではあるけど、イマイチ掴めず解けなかった……。

## 問題

数列に対する最小公倍数は、その数列の全要素で割り切れる値とする。
例えば、$lcm(1,2,3,4,5)=60$となる。

ある自然数Nがあるとき、$lcm(N+1, N+2, ..., M) = lcm(1,2,3,...,M)$となる最小のMを求めよ。
なお、このようなMは必ず存在する。

$$
1 \leq N \leq 10^6
$$

## 考え方

$M = 2N$の時、明らかに成り立つ。
また、$1, 2, ..., N$のうちの素数が、$N+1, N+2, ..., M$の約数に全て出現する時、条件を満たす**場合ががある**。

例えば、サンプルの5つ目。
1～42にある素数は、$2, 3, 5, ..., 41$。
$42 = 2 \cdot 3 \cdot 7$なので、$N+1, N+2,...,M$に$2,3,7$を約数として持つのであれば、$M < 42 \times 2$で十分である。
$44 = 2^2 \cdot 11$、$48 = 2^4 \cdot 3$、$49 = 7^2$などが$2,3,7$を含むので、$M=84$である必要はない。
この場合、1つ前の$41$が素数なので、これを2倍した$M = 82$でよい。

ただし、これだけでは成り立たない場合がある。

サンプルの3つ目。
$N = 2^2$なので、$M=6$ -> $lcm(5, 6)$では4の倍数にならないのでダメ。
$M=8$であれば、$8 = 2^3$なので条件を満たす。

$N=10$を考える。
$N = 2 \cdot 5$を2倍すると$2^2 \cdot 5$になるが、$N+1$から$3^2$を2倍した$3^2 \cdot 2$までの値に、必要な約数は全て入っている。

というわけで、以下の手順で答えが求まる。

1. N以下の素数を列挙し、それぞれを累乗してNを超えない中での最大値を記録
2. それぞれを2倍
3. その中の最大が答え

N=10の場合であれば、素数は [2,3,5,7]、Nを超えないギリギリは、[$2^3, 3^2, 5, 7$]。
それぞれを倍し、[$2^4, 2\cdot3^2, 5\cdot2, 7\cdot2$] = [16, 18, 10, 14]。
この中で最大の18が答えになる。

ちなみに、素数判定はエラトステネスの篩でやればよい。

```C++
class MissingLCM {
    public:

    // a^k
    ll fpow(ll a, ll k) {
        ll res = 1ll;
        ll x = a;
        while (k != 0) {
            if ((k & 1) == 1)
                res = (res * x);
            x = (x * x);
            k >>= 1;
        }
        return res;
    }

    int prime[MAX];
    bool is_prime[MAX+1];

    int sieve(int n) {
        int p = 0;
        rep(i, n+1) is_prime[i]= true;
        is_prime[0]= is_prime[1] = false;
        REP(i, 2, n+1) {
            if(is_prime[i]) {
                prime[p++] = i;
                for(int j=2*i; j<=n; j+=i)
                    is_prime[j] = false;
            }
        }
        return p;
    }

    int getMin(int N) {

        sieve(MAX-1);

        vi maxpow(N+1, 0);
        maxpow[0] = maxpow[1] = 1;

        REP(i, 2, N+1){
            if(is_prime[i]) {
                REP(j, 2, 32){
                    if( fpow(i, j) > N ) {
                        maxpow[i] = fpow(i, j - 1);
                        break;
                    }
                }
            }
        }

        int t = *max_element(all(maxpow));

        return (t*2);
    }

};
```

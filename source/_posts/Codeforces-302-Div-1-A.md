title: "Codeforces #302 (Div. 1) A"
date: 2015-05-15 23:42:21
tags:
- esplo77
- Codeforces
- C++11
---

# A. Writing Code

## 問題

m行のコードを書く必要がある。
現場にはn人のプログラマーがいて、$i$番目のプログラマーは、毎行$a_i$個のバグを生む。
バグをb個以下に抑えるとき、それぞれのプログラマーが書く行数のパターンはいくつあるか。
答えはmodでの剰余を出力せよ。

$$
1 \leq n, m \leq 500 \\\
0 \leq b \leq 500 \\\
1 \leq mod \leq 10^9 + 7 \\\
0 \leq a_i \leq 500 \\\
3ms
$$

## 考え方

## 基礎
「総数を求める」「最大コストの制限がある」ということで、DPだと当たりを付ける。

i番目までのプログラマーが、j行を書き、k個のバグを生み出した、と置く。
この場合、漸化式は以下のようになる。

$$
\begin{aligned}
dp[i][j][k] & = & dp &[i-1]&[j]&[k] \\\
  & + & dp&[i-1]&[j-1]&[k-a[i-1]] \\\
  & + & dp&[i-1]&[j-2]&[k-2 \cdot a[i-1]] \\\
  & + & & & ... & \\\
  & + & dp&[i-1]&[j-t]&[k-t \cdot a[i-1]]
\end{aligned}
$$
（tは$0 \leq t; 0 \leq k-t \cdot a[i-1]$を満たす間回す）
（左に詰めて欲しい）

各プログラマーが何行書くか分からないので、このようになる。

## 速度
上記DPだと、$O(n \cdot m \cdot b \cdot m)$となり、$10^{10}$くらいのループになってしまう。
これではさすがのC++も間に合わない。

ということで、漸化式をじっくり見てみると、以下の式が見えてきたりする。


$$
\begin{aligned}
dp[i][j-1][k-a[i-1]] & = & dp &[i-1]&[j-1]&[k-a[i-1]] \\\
  & + & dp&[i-1]&[j-2]&[k-2 \cdot a[i-1]] \\\
  & + & & & ... & \\\
  & + & dp&[i-1]&[j-t]&[k-t \cdot a[i-1]]
\end{aligned}
$$

おや、どこかで見たことがある……。
というわけで、最初の式は以下のように変形。

$$
\begin{aligned}
dp[i][j][k] & = & dp &[i-1]&[j]&[k] \\\
            & + & dp &[i]&[j-1]&[k-a[i-1]]
\end{aligned}
$$

これで$O(n \cdot m \cdot b)$で$10^8$程度。
C++で3msあれば間に合う回数になった。

## メモリ

これだけでは終わらないのがDiv1。
dpの各要素は最低でも$10^9+7$を保持できないといけない（modが最大$10^9+7$なので）ため、4byte必要。
$4 \cdot 500 \cdot 500 \cdot 500 = 4 \cdot 125 \cdot 10^6 = 5 \cdot 10^8$ なので、およそ500MB必要で溢れてしまう。

再度漸化式を凝視すると、dp[i]はdp[i-1]のみに依存しているので、iの所は2を交互に使えば十分。
これで、2MBになりメモリもOK。

## コード

```C++
const int MAXN = 510;
int dp[2][MAXN][MAXN];
int a[MAXN];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n, m, b, mod;
    cin >> n >> m >> b >> mod;
    rep(i, n)
        cin >> a[i];

    rep(i, 2) rep(j, MAXN) rep(k, MAXN) dp[i][j][k] = 0;
    dp[0][0][0] = 1;

    REP(orig_i, 1, n + 1) {
        int i = orig_i & 1;
        rep(j, m + 1) {
            rep(k, b + 1) {
                dp[i][j][k] = dp[i ^ 1][j][k];
                if (0 <= j - 1 && 0 <= k - a[orig_i - 1]) {
                    dp[i][j][k] += dp[i][j - 1][k - a[orig_i - 1]];
                }
                dp[i][j][k] = (dp[i][j][k] % mod);
            }
        }
    }

    int res = 0;
    rep(i, b + 1)
        res = (res + dp[n % 2][m][i]) % mod;
    cout << (res % mod) << endl;

    return 0;
}
```

## 結果
436ms, 2100KB

問題文「プログラマーがバグを生むなら、みんな死ぬしかないじゃない！」


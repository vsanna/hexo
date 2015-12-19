title: "Codeforces #326 Div.2 D"
date: 2015-12-20 01:18:39
tags:
- esplo77
- Codeforces
- C++11
---

# D. Duff in Beach
[問題文](http://codeforces.com/contest/588/problem/D)

## 考え方
[ここを見た](http://kmjp.hatenablog.jp/entry/2015/10/17/1030)
問題がとにかくややこしいが、条件を整理すろと、以下の処理を行えば良いことが分かる。

1. width個の要素を持つ配列を作る時、その組み合わせ数を求める
1. 組み合わせの開始位置をスライドして、全パターンを求める

### 組み合わせの数を求める
組み合わせはDPで求める事ができる。
wを長さ、vを要素の値、
dp[w][v] := 長さwで最後の要素がvの時の組み合わせ数
とすると、
$dp[w+1][v] += dp[w][v]$
を全要素に対し行った後、累積和を取ると組み合わせ数が求まる。
$dp[w+1][i-1] += dp[w+1][i] (1 \leq i < k)$

例えば、n=3の配列 [2,3,1] があり、dp[1]の各要素を1にしたとする（長さ1はそれぞれ1パターン）。
1の処理を行うと、
dp[2][2] = dp[2][3] = dp[2][1] = 1
となり、2の処理を行い、
dp[2][1] = 1 (1-1)
dp[2][2] = 2 (1-2, 2-2)
dp[2][3] = 3 (1-3, 2-3, 3-3)
となる。

同様に、長さkまで処理を行うと、それぞれの要素で組み合わせ数が求まる。

### パターン数（答え）を求める
長さがwの時、$(l/n)$の配列nのブロックがあるため、$(l/n)-w+1$の開始位置がある。
これをdp[w]の各要素で掛けあわせ、結果を足し合わせると答えになる。
ただし、$(l/n)$のブロックからはみ出る部分があるため、その場合は開始位置を+1する。

### その他
- 座標圧縮を事前に行なうと、上記処理が簡単に行える。
- DPはそのままだとMLEするので、いつもの配列2つを入れ替える方法でループする。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    ll n, l, k;
    cin >> n >> l >> k;
    vi _a(n), a(n);
    rep(i, n) {
        cin >> a[i];
        _a[i] = a[i];
    }

    // 圧縮する
    sort(all(_a));
    _a.erase(unique(all(_a)), _a.end());
    rep(i, n)
        a[i] = lower_bound(all(_a), a[i]) - _a.begin();

    // 1ループ中の要素ごとの組み合わせ数
    vector<vector<ll> > dp(2, vector<ll>(n, 1));

    ll res = 0;
    vector<ll>& cur = dp[0];
    vector<ll>& next = dp[1];

    // 取り出す個数
    REP(width, 1, k + 1) {
        fill(all(next), 0);

        // dp
        rep(i, n) {
            next[a[i]] += cur[a[i]];
            next[a[i]] %= MOD;

            // はみ出る分を考慮
            int add = (int) (i < l % n);

            // スライドした分を掛ける
            ll length = l / n + add;
            if (length >= width) {
                res += (cur[a[i]] * ((length+1-width)%MOD)) % MOD;
                res %= MOD;
            }
        }
        // 累積和をとる
        REP(i, 1, n) {
            next[i] += next[i-1];
            next[i] %= MOD;
        }

        swap(cur, next);
    }

    cout << res << endl;

    return 0;
}
```
title: "Codeforces #338 Div.2 A-D"
date: 2016-01-11 20:42:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Bulbs
[問題文](http://codeforces.com/contest/615/problem/A)

### 考え方
出現した数が1-Nを網羅しているかを判定すれば良い。
setに入れてサイズを比較して終わり。

```C++
int main() {
	int n,m;
    cin >> n>> m;
    set<int> ans;
    rep(i, n) {
        int t;
        cin >> t;
        rep(j, t) {
            int u;
            cin >> u;
            ans.insert(u);
        }
    }
	if(ans.size() != m)
        cout << "NO" << endl;
    else
        cout << "YES" << endl;
    return 0;
}
```

## B. Longtail Hedgehog
[問題文](http://codeforces.com/contest/615/problem/B)

### 考え方
readforces。
spineも$u<v$が成り立つ必要があると思ってWAを食らった。
せめてサンプルに含めて欲しかったな……。

最長の長さをDPで求める。
u-vというエッジの時、vが小さい順にDPすればDAGになる。
intではあふれるケースがあるので、long longにすることだけ注意。

```C++
int main() {
    int n, m;
    cin >> n >> m;

    vector<pii> list(m);
    vi in(n, 0), dp(n, 0);

    rep(i, m) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        if (u > v) swap(u, v);
        list[i] = mp(v, u);
        in[v]++;
        in[u]++;
    }
    sort(all(list));

    rep(i, m) {
        pii t = list[i];
        // invert
        int u = t.second, v = t.first;
        chmax(dp[v], dp[u] + 1);
    }

    ll ans = 0;
    rep(i, n)
        chmax(ans, (ll)in[i] * (dp[i] + 1));
    cout << ans << endl;

    return 0;
}
```

## C. Running Track
[問題文](http://codeforces.com/contest/615/problem/C)

### 考え方
コンテストではZ Algorithmを使ってゴリ押ししたが、もっと簡単に求まる。

#### greedy
ひとまず逆順は置いて考える。
tの各位置～末尾（t[i:M]）と、sの部分文字列でなるべく一致するものを貪欲に使ってゆけば良い。
s=abc$cd, t=abcdという文字列があったとして、部分文字列が一致すれば良いため、abc -> dと使うことも、ab -> cdと使うことも可能である。
そのため、最長のものを使っても後に損をしないので、貪欲が使える。

#### prefix
一致している文字数を求めるのにいちいち部分文字列を持ってきてはTLEするので、それぞれの文字列のprefixで求める。

後々都合が良いため、tを先に持ってきた以下のDP表を埋めてゆく。
dp[ti][si] := t[ti:M]とs[si:N]の最長一致プレフィックス

最長一致を普通に求めるとまたTLEしてしまうので、文字列の後ろからDPをすると効率的に求まる。

逆順にしたDP表も求め、2つをマージして表を辿れば答えが求まる。
どの範囲を使ったかもいるので、保存して後で出力する（この出力、必要だったのか……？最短回数を求める問題で十分だったのでは）.

```C++
string from, to;
vvi update() {
    vvi lcp(to.size()+1, vi(from.size()+1, 0));
    for(int i=to.size()-1; i>=0; --i) {
        for(int j=from.size()-1; j>=0; --j) {
            if(to[i] == from[j])
                lcp[i][j] = lcp[i+1][j+1] + 1;
        }
    }
    return lcp;
}

int main() {
    cin >> from >> to;

    vvi lcpn = update();
    reverse(all(from));
    vvi lcpr = update();

    vi dp(to.size(), 0);
    vector<pii> dp_range(to.size());
    rep(i, to.size()) {
        auto it_mx_n = max_element(all(lcpn[i]));
        auto it_mx_r = max_element(all(lcpr[i]));
        dp[i] = max(*it_mx_n, *it_mx_r);
        if(*it_mx_n >= *it_mx_r) {
            int dist = distance(lcpn[i].begin(), it_mx_n);
            dp_range[i] = mp(dist+1, dist + *it_mx_n);
        }
        else {
            int dist = distance(it_mx_r, lcpr[i].end());
            dp_range[i] = mp(dist-1, dist - *it_mx_r);
        }
    }

    int ans = 0;
    vector<pii> route;
    for(int i=0; i < (int)to.size(); ) {
        if(dp[i] == 0) {
            cout << -1 << endl;
            return 0;
        }
        ans++;
        route.push_back(dp_range[i]);
        i += dp[i];
    }

    cout << ans << endl;
    tr(it, route)
        cout << it->first << " " << it->second << endl;

    return 0;
}
```


## D. Multipliers
[問題文](http://codeforces.com/contest/615/problem/D)

### 考え方
modの世界に関しての前提知識が結構必要。

#### 解き方
問題を色々変形していると、結局以下の値を求めれば良いことが分かる。

各素数を$p_i$、それぞれの個数を$n_i$、素数の種類が$k$個あるとする。

各iについて、p[i]の$n$乗までのそれぞれを$\prod_{j=0}^{k-1} (n_j+1)$ (ただし、$i \neq j$)乗し、それを掛けあわせたもの

イマイチ分かりにくいので、以下の例で示す。
{2, 2, 2, 3, 3, 4, 4}のときを考えると、
p={2, 3, 4}, n={3, 2, 2}, k=3 ($|p|$)となる。

i=1について、$p_1$の3乗までとは、2, 4, 8となる。
これのそれぞれについて、$n_1$以外を+1し掛けあわせた数($= (2+1) \times (2+1) = 9$) 乗する。
結果は、$2^9 \times 4^9 \times 8^9$となる。
これをそれぞれのiについて行い、その結果同士をかけ合わせると答えになる。
なお、$n_i$に1を加えているのは、その集合から取らないパターン（0乗）を加えていることを表している。

これをそのまま実装すると、勿論値が大きすぎるのと、TLEする。
そのため、以下の工夫が必要になる。

#### 高速に計算
まず、累乗は繰り返し二乗法を使う。

また、以下の部分で時間が掛かってしまうので、式変形を行う。
$2^9 \times 4^9 \times 8^9$

$2^v$ は、2をv回掛ける。
$4^v$ は、2を2v回掛ける。合計で3v回。
$8^v$ は、2を3v回掛ける。合計で6v回。

というわけで、$p$の$n$乗まで足し合わせるのは、
pを$v \times \sum_{i=1}^n i = v \times \frac{n \times (n+1)}{2}$乗すれば求まる。

#### mod関連
罠ポイントとして、mod pの指数をmod pで計算するとダメ、というものがある。
フェルマーの小定理では、
$a^{p-1} \equiv 1 (mod p)$
なので、指数部は$mod p-1$で計算しないといけない。

今回の場合だと、指数部は$mod 10^9 + 6$で計算する。
これは素数ではないので、フェルマーの小定理で逆元が求まらない。
そのため、[0:i-1]の累積積配列、[i+1:n]の累積積配列をそれぞれ用意し、iを含まない積を計算すれば良い。

```C++
int main() {
    int m;
    cin >> m;

    map<int, int> cnt;
    rep(i, m) {
        int t;
        cin >> t;
        cnt[t]++;
    }
    // mapだと扱いにくいので
    vector<pii> vp;
    tr(it, cnt)
        vp.push_back(*it);

    // 指数部分のための累積積
    // prod_pref[i] = v[0]*v[1]*...*v[i-1]
    vll prod_pref(vp.size() + 1, 1);
    // prod_suff[i] = v[i+1]*v[i+2]*...*v[n-1]
    vll prod_suff(vp.size() + 1, 1);
    REP(i, 1, vp.size()+1)
        prod_pref[i] = (prod_pref[i-1] * (vp[i-1].second + 1)) % (MOD - 1);
    for (int i = vp.size() - 1; i >= 0; --i)
        prod_suff[i] = (prod_suff[i + 1] * (vp[i].second + 1)) % (MOD - 1);

    ll ans = 1;
    rep(i, vp.size()) {
        int v = vp[i].first;
        int p = vp[i].second;

        // vを何回掛ける必要があるか
        ll times = (((ll)(p + 1) * p) / 2) % (MOD - 1);
        ll rem = (prod_pref[i] * prod_suff[i+1]) % (MOD - 1);
        ll exp = (times * rem) % (MOD - 1);
        ans *= fpow(v, exp, MOD);
        ans %= MOD;
    }

    cout << ans << endl;

    return 0;
}
```

## まとめ
参加者は3673人。

- A: 3464/3593
- B: 778/1814
- C: 294/614
- D: 208/1572
- E: 83/161

B問題は問題文がわかりにくく、C問題の難易度が高く、D問題もミスするポイントが多いという辛いセットだった。
指数は$mod p-1$しないといけない、というのは初めて気付いたので、今後注意しよう。


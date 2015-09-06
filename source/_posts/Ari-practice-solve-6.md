title: "Ari-practice solve #6"
date: 2015-08-31 15:58:53
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (6)
The DP.

## POJ1742 - Coins
蟻本の通りに解けば良い。
が、地力でやると、DP内のループが遅くてTLEした。
時間制限が厳しい……。

次のDP配列に対して処理を行ったりと、少しトリッキーな気がする。
これをサクッと書けるようになりたい。

```C++
const int MAX_N = 101;
const int MAX_M = 100001;
int a[MAX_N];
int c[MAX_N];
int dp[2][MAX_M];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n, m;
    while (true) {
        scanf("%d", &n);
        scanf("%d", &m);
        if (n == 0 && m == 0)
            break;

        memset(a, 0, sizeof(a));
        memset(c, 0, sizeof(c));
        rep(i, n) scanf("%d", &a[i]);
        rep(i, n) scanf("%d", &c[i]);

        // dp[i][j] := i番目まででjを作った時、iの余り
        memset(dp, -1, sizeof(dp));
        dp[0][0] = 0;

        rep(_i, n) {
            int i = _i % 2;
            int ni = (i + 1) % 2;

            rep(j, m + 1) {
                if (dp[i][j] >= 0) {  // 使わないー
                    dp[ni][j] = c[_i];  // c[i+1]は常に最大なので、max不要
                }
                else if (j - a[_i] >= 0 && dp[ni][j - a[_i]] >= 1) {  // 使うー。配るー
                    dp[ni][j] = dp[ni][j - a[_i]] - 1;  // c[i+1]は常に最大なので、max不要
                }
            }
        }

        int res = 0;
        REP(i, 1, m + 1)
            res += (dp[n % 2][i] >= 0) ? 1 : 0;

        printf("%d\n", res);
    }
    
    return 0;
}
```

## POJ3046 - Ant Counting
蟻本通りではあるが、蟻本の変形が難しい。
最後に引き算が必要な場合は、$j > a[i]$の時。

書き下すと分かるが、左辺は
$$
dp[i][j] + dp[i][j-1] + ... + dp[i][j-a[i]]
$$
となり、右辺の1つ目は
$$
dp[i][j-1] + dp[i][j-2] + ... + dp[i][j-a[i]] + dp[i][j-1-a[i]]
$$
となるので、式の通り調整すると正しくなる。

思いつかん。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int M = 1000000;

    int T, A, S, B;
    cin >> T >> A >> S >> B;

    vi ant(T, 0);
    rep(i, A) {
        int t;
        cin >> t;
        ant[t-1]++;
    }

    // dp[i+1][j] := i番目までの品物をj個選ぶ組み合わせの総数
    vector< vi > dp(T+1, vi(A+1, 0));
    // 1つも選ばない
    rep(i, T+1) dp[i][0] = 1;

    rep(i, T) {
        REP(j, 1, A+1) {
            if(j-1-ant[i] >= 0) {
                dp[i+1][j] = (dp[i+1][j-1] + dp[i][j] - dp[i][j-1-ant[i]] + M) % M;
            }
            else {
                dp[i+1][j] = (dp[i+1][j-1] + dp[i][j]) % M;
            }
        }
    }

    int res = 0;
    REP(i, S, B+1)
        res = (res + dp[T][i]) % M;

    cout << res << endl;

    return 0;
}
```


## POJ3181 - Dollar Dayz
個数制限なしナップサック問題。
ただ、桁数がえげつないので多倍長計算が必要。
C++で用意するのが面倒なので、省略。


## POJ1065 - Wooden Sticks ☆
まずは、[このページ](https://topcoder.g.hatena.ne.jp/cafelier/20121011)で言葉の定義を学習。

- 独立パス被覆
- 独立点集合（反鎖）
- Dilworthの定理

第一要素でソートし、第二要素を左から右に、小さい数から大きい数に辺をはるとDAGになる。
求めたい答えは、このDAGの |最小独立パス被覆|。
Dilworthの定理から、これは |最大独立点集合| に等しい。

この場合の最大独立展集合の最大値は、|減少数値列の最長|。
というわけで、蟻本で見たことのあるDPになった。

とても勉強になる問題だったので、タイトルに☆を付けておいた。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int T;
    cin >> T;

    while (T--) {
        int N;
        cin >> N;
        vector<pii> sticks(N);
        rep(i, N)
            cin >> sticks[i].first >> sticks[i].second;

        sort(all(sticks));

        vi dp(N+1, INF);
        rep(i,N) {
            int minusw = sticks[i].second * -1;
            *lower_bound(all(dp), minusw) = minusw;
        }

        cout << distance(dp.begin(), lower_bound(all(dp), INF)) << endl;
    }

    return 0;
}
```


## POJ1631 - Bridging signals
問題文が長いだけ。
さっきの問題が解けた人は、一瞬で解ける問題。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int TEST;
    cin >> TEST;
    while(TEST--) {
        int p;
        cin >> p;

        vi blocks(p, 0);
        rep(i, p)
            cin >> blocks[i];

        vi dp(p, INF);
        rep(i, p) {
            *lower_bound(all(dp), blocks[i]) = blocks[i];
        }
        cout << lower_bound(all(dp), INF) - dp.begin() << endl;
    }
    
    return 0;
}
```


## POJ3666 - Making the Grade
[ここを参考に書いた。](http://takapt0226.hatenablog.com/entry/2013/01/06/052436)
漸化式がぱっと浮かばない。修行あるのみ。

dpをvector< vector<int> >で取ったらTLEした。
そんなに速度が変わるものなんだろうか……。


```C++

int dp[2001][2001];

int solve(int N, vi& A, vi& h) {
    // dp[i+1][j] := i を　h[j] にする最小コスト
    rep(i,N+1) rep(j,N+1) dp[i][j] = INT_MAX;
    dp[0][0] = 0;

    rep(i, N) {
        int prev_min = INT_MAX;  // そこまでの最小dp[i][j]
        rep(j, N) {
            prev_min = min(prev_min, dp[i][j]);
            dp[i+1][j] = min(dp[i+1][j], prev_min + abs(A[i]-h[j]));
        }
    }

    return *min_element(dp[N], dp[N]+N+1);
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    vi A(N), h(N);
    rep(i,N) {
        cin >> A[i];
        h[i] = A[i];
    }

    // ascending
    sort(all(h));
    int res = solve(N, A, h);

    // descending
    sort(all(h), greater<int>() );
    res = min(res, solve(N, A, h));

    cout << res << endl;

    return 0;
}
```

## POJ2392 - Space Elevetor
普通の複数個選べるナップサック問題。
癒される難易度。
なお、配列の使い回しをするのを忘れていたが、しなくてもACだった。
POJにしては制限が緩いのかも。

```C++

const int MAXH = 40001;
int dp[400+1][MAXH];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int K;
    cin >> K;

    pair<int, pii> blocks[K];
    rep(i,K) {
        pii t;
        cin >> t.first >> blocks[i].first >> t.second;
        blocks[i].second = t;
    }

    sort(blocks, blocks+K);

    // dp[i+1][j] := i番目までのブロックで高さjを作った時、残りのc[i]
    memset(dp, -1, sizeof(dp));
    dp[0][0] = blocks[0].second.second;

    rep(i, K) {
        int hlim = blocks[i].first;
        pii t = blocks[i].second;
        int a = t.first;
        int c = t.second;
        rep(j, MAXH) {
            if(j > hlim)
                continue;

            if(dp[i][j] >= 0)
                dp[i+1][j] = c;
            else if(j-a < 0 || dp[i+1][j-a] <= 0)
                dp[i+1][j] = -1;
            else
                dp[i+1][j] = dp[i+1][j-a] - 1;
        }
    }

    int mx = 0;
    rep(i, MAXH) {
        if(dp[K][i] > -1)
            mx = i;
    }

    cout << mx << endl;

    return 0;
}
```


## POJ2184 - Cow Exhibition
マイナスをどう取り扱うか、の問題。
でかい配列を作って回避した。
問題自体は基本的なナップサックなので簡単。
最後のループで終了値を MAXV - ZERO になぜかしていてずっと悩んでいた。

```C++

const int MAXV = 100 * 1000 * 2 + 10;
const int ZERO = 100 * 1000 + 2;
int dp[2][MAXV];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    pii cows[N];
    rep(i,N) {
        cin >> cows[i].first >> cows[i].second;
    }

    // dp[i+1][j] := i番目まででTS = jとなったときのTF
    rep(i,2) {
        fill(dp[i], dp[i]+MAXV, -INF);
    }
    dp[0][ZERO] = 0;

    int* cur = dp[0];
    int* next = dp[1];
    rep(i,N) {
        fill(next, next+MAXV, -INF);
        int s = cows[i].first;
        rep(j, MAXV) {
            if(cur[j] != -INF) {
                if (0 <= j + s && j + s < MAXV)
                    next[j + s] = max(next[j + s], cur[j] + cows[i].second);
                next[j] = max(next[j], cur[j]);
            }
        }
        swap(cur, next);
    }

    int res = 0;
    int modN = N % 2;
    REP(i, ZERO, MAXV) {
        if(dp[modN][i] >= 0)
            res = max(res, dp[modN][i] + (i-ZERO));
    }
    cout << res << endl;

    return 0;
}
```


## まとめ
長い辛いDPだった。
その甲斐あってか、苦手意識がそれなりに無くなった気がする。
ぱっと漸化式が立てられるレベルに至っていないうえ、様々な未知のパターンがあると思うので、色んなDPに触れて精進してゆきたい。

それにしてもPOJはどこでミスっているのか分からないので練習には辛い。
コドフォみたいにテストケースが見えるようになればいいのに……。

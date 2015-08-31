title: "Ari-practice solve #5"
date: 2015-08-25 23:18:57
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (5)
ついにDPへ。

## POJ3176 - Cow Bowling
配るDPにした。
なんか汚いけど、まぁいいか……。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    int pin[N][N];
    memset(pin, -1, sizeof(pin));
    int dp[N][N];
    memset(dp, 0, sizeof(dp));

    rep(i,N) {
        rep(j,i+1) {
            cin >> pin[i][j];
        }
    }

    dp[0][0] = pin[0][0];
    rep(i,N-1) {
        rep(j,i+1) {
            dp[i+1][j] = max(dp[i+1][j], dp[i][j] + pin[i+1][j]);
            dp[i+1][j+1] = max(dp[i+1][j+1], dp[i][j] + pin[i+1][j+1]);
        }
    }

    int res = 0;
    rep(j,N) {
        res = max(res, dp[N-1][j]);
    }
    
    cout << res << endl;
    
    return 0;
}
```

## POJ2229 - Sumsets
気付かなかった。
確かにそうなるけど、パッと思いつかんなぁ……。
実装は超簡単。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    int dp[N+1];
    memset(dp, 0, sizeof(dp));
    dp[0] = 1;

    REP(i,1,N+1) {
        dp[i] = dp[i-1];
        if(i%2==0) dp[i] += dp[i/2];
        dp[i] %= 1000000000;
    }

    cout << dp[N] % 1000000000 << endl;

    return 0;
}
```

## POJ2385 - Apple Catching
配った。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int T, W;
    cin >> T >> W;

    int dp[2][W+5][T+5];
    memset(dp, -1, sizeof(dp));

    int ap[T];
    rep(i,T) cin >> ap[i];

    dp[0][0][0] = 0;

    rep(i, T+1) {
        rep(w, W+1) {
            rep(pos, 2) {
                if(dp[pos][w][i] == -1)
                    continue;

                int t1=0, t2=0;
                if(ap[i]-1 == pos)  t1=1;
                else t2 = 1;

                // no movement
                dp[pos][w][i+1] = max(dp[pos][w][i+1], dp[pos][w][i] + t1);
                // move
                dp[(pos+1) % 2][w+1][i+1] = max(dp[(pos+1) % 2][w+1][i+1], dp[pos][w][i] + t2);
            }
        }
    }

    int mx = 0;
    rep(pos, 2) {
        rep(w, W+1) {
            mx = max(dp[pos][w][T], mx);
        }
    }

    cout << mx << endl;

    return 0;
}
```

## POJ3616 - Milking Time
座標圧縮さえすれば、あとは単純にDPすればよい。
どうにもコードが長くなってしまう気がする……。

```C++

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M, R;
    cin >> N >> M >> R;

    vector< pair<pii, int> > works(M);
    vi times;
    rep(i,M) {
        int s,e,ef;
        cin >> s >> e >> ef;
        works[i] = mp(mp(s,min(N,e+R)), ef);
        times.push_back(s);
        times.push_back(min(N,e+R));
    }
    times.push_back(N);
    sort(all(works));

    sort(all(times));
    times.erase(unique(all(times)), times.end());

    // 座標圧縮
    map<int, int> conv;
    rep(i, times.size())
        conv.insert(mp(times[i], i));

    vi dp(times.size(), 0);
    rep(i, M) {
        pii time = works[i].first;
        int eff = works[i].second;
        int sidx = conv[time.first];
        int eidx = conv[time.second];

        REP(j, eidx, dp.size()) {
            dp[j] = max(dp[j], dp[sidx] + eff);
        }
    }

    cout << dp[conv[N]] << endl;

    return 0;
}
```


## POJ3280 - Cheapest Palindrome
区間DPなるものらしい。
あまりやったことがなく、手こずった。
2000*2000のint型配列をスタックに持てなかったので散々REを食らった。

```C++

// dp[i][j] := 文字列の [i,j) をPalindromeにするためのコスト
int dp[2005][2005];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    string str;
    cin >> str;

    map<char, int> cost;
    rep(i,N) {
        char c;
        int c1, c2;
        cin >> c >> c1 >> c2;
        // 小さい方だけでOK
        cost[c] = min(c1, c2);
    }

    // 空文字列、1文字はコスト0
    rep(i,M) {
        fill(dp[i], dp[i]+M+1, INF);
        dp[i][i] = dp[i][i+1] = 0;
    }

    // 開始位置のループが内側
    rep(len,M) {
        rep(i,M) {
            int j = i + len;
            if(j > M+1)
                continue;
            // 左に区間を伸ばす。左を削除 or 右を追加
            if(i > 0)
                dp[i-1][j] = min(dp[i-1][j], dp[i][j] + cost[str[i-1]]);
            // 右に区間を伸ばす。左を追加 or 右を削除
            if(j < M)
                dp[i][j+1] = min(dp[i][j+1], dp[i][j] + cost[str[j]]);
            // 左右に伸ばす。両方同じならコスト0
            if(i > 0 && j < M && str[i-1] == str[j])
                dp[i-1][j+1] = min(dp[i-1][j+1], dp[i][j]);
        }
    }

    cout << dp[0][M] << endl;

    return 0;
}
```


## まとめ
DPの基礎。
DPとしてはシンプルなものばかりだが、そうなる理由が意外と難しかったり。
基礎練には丁度よさそう。

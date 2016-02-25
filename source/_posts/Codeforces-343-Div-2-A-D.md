title: "Codeforces #343 A-D"
date: 2016-02-24 00:13:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Far Relative’s Birthday Cake
[問題文](http://codeforces.com/contest/629/problem/A)

### 考え方
左上から右下へ辿ってゆき、Cを見つけたら更に右と下にある個数を数えることで、$O(n^3)$で漏れ無くペアを列挙できる。

### コード

```C++
int main() {
    int n;
    cin >> n;

    vector<vector<bool> > mat(n, vector<bool>(n, false));
    rep(i,n ){
        string str;
        cin >> str;
        rep(j, n) {
            if(str[j] == 'C') mat[i][j] = true;
        }
    }

    ll ans = 0;
    rep(i, n) {
        rep(j, n) {
            if(mat[i][j]) {
                REP(k, i+1, n)
                    if(mat[k][j]) ans++;
                REP(k, j+1, n)
                    if(mat[i][k]) ans++;
            }
        }
    }

    cout << ans << endl;
    
    return 0;
}
```


## B. Far Relative’s Problem
[問題文](http://codeforces.com/contest/629/problem/B)

### 考え方
$n$と$b$が小さいため、日付に対して全探索し、男女同数になる中で最大を探せばよい。
$O(nb)$

### コード

```C++
int main() {
    int n;
    cin >> n;
    vector< pair< pii, bool > > people(n);
    rep(i, n) {
        char c;
        cin >> c;
        pii t;
        cin >> t.first >> t.second;
        people[i] = mp(t, c == 'M');
    }

    int ans = 0;
    REP(day, 1, 366+1) {
        pii mf = mp(0, 0);
        rep(i, n) {
            pii date = people[i].first;
            if(date.first <= day && day <= date.second) {
                if(people[i].second) mf.first++;
                else mf.second++;
            }
        }
        chmax(ans, min(mf.first, mf.second) * 2);
    }

    cout << ans << endl;
    
    return 0;
}
```

## C. Famil Door and Brackets
[問題文](http://codeforces.com/contest/629/problem/C)

### 考え方
まず、"("の数-")"の数を使って括弧のバランスを表す。
dp[i][j] := 長さiとバランスj
とし、それぞれのパターン数を求める。
「（を追加する」、「）を追加する」のいずれかを行う（以下の漸化式）ことで、値を求めることができる。

$dp[i+1][j] = dp[i][j-1] + dp[i][j+1]$

次に、sの括弧を数え、左側に追加する必要のある"("と、右側に追加する必要のある")"の数を求める。

最後に、「左側の長さ」「バランス」で左右に追加する文字列を決め、パターン数を計算する。
$O((n-m)^2)$。

### コード

```C++
int main() {
    int n, m;
    cin >> n >> m;
    string s;
    cin >> s;

    int lo = 0;   // 左で開く必要のある括弧の数
    int rc = 0;   // 右で閉じる必要のある括弧の数
    rep(i, s.size()) {
        if (s[i] == '(') rc++;
        else rc--;
        chmin(lo, rc);
    }
    lo *= -1;

    // dp[i][j] := 長さiで |(| - |)| = j のときのパターン数
    vvll dp(n - m + 1, vll(n - m + 1, 0));
    dp[0][0] = 1;
    REP(i, 1, n - m + 1) {
        rep(j, i + 1) {     //  |(| - |)|
            dp[i][j] = dp[i - 1][j + 1];
            if (j > 0)
                dp[i][j] += dp[i - 1][j - 1];
            dp[i][j] %= MOD;
        }
    }

    ll ans = 0;
    rep(i, n - m + 1) {
        rep(left, i + 1) {
            if (left >= lo) {   // 開くべき数以上
                if (0 <= left + rc && left + rc <= n - m) {
                    // 左側 * 右側 のパターン
                    ans += (dp[i][left] * dp[n - m - i][left + rc]) % MOD;
                    ans %= MOD;
                }
            }
        }
    }

    cout << ans << endl;

    return 0;
}
```

## D. Babaei and Birthday Cake
[問題文](http://codeforces.com/contest/629/problem/D)

### 考え方
以下の3つの条件を満たすよう、答えを求める必要がある。
i番目のケーキの体積をv[i]、j番目のケーキの上にi番目を乗せるとすると、

1. $j < i$
2. $v[j] < v[i]$
3. 体積合計の最大

これらの条件を満たすため、以下のDPを求める。
dp[最後に乗せたケーキの体積] := ケーキの体積の合計
DPの埋め方は以下（最初にdp[0]=0、それ以外は-1とする）。
dp[v[i]] := max(dp[0]～dp[v[i]-1]) + v[i]

これを0～nまで順に行うことで、上記3つの条件が満たせる。

1. 0～nへ順番に行うことで、iより大きい要素はまだ表に無い
2. max(dp[0]～dp[v[i]-1])で求まる
3. 埋めたあと、max(dp[0]～dp[v[n-1]])で求まる

原理は上記の通りだが、dpの要素数が大きくなりすぎるため、座標圧縮を行い$10^5$とする。
また、2の処理で$O(n)$かかってしまうため、セグメント木を使って$O(logn)$にし、トータルで$O(nlogn)$に抑える。


### コード

```C++
const static double PI = 4*atan(1.0);
template <typename T>
class SegTree {
    int n;
    vector<T> dat;

public:
    SegTree(int _n) {
        n = 1;
        while(n < _n) n *= 2;
        dat.resize(2*n-1);
        fill(all(dat), -1);
    }

    void update(int k, T a) {
        k += n - 1;
        dat[k] = a;
        while(k > 0) {
            k = (k-1) / 2;
            dat[k] = max(dat[k*2+1], dat[k*2+2]);
        }
    }

    T _query(int a, int b, int k, int l, int r) {
        if(r <= a || b <= l) return -1;

        if(a <= l && r <= b) return dat[k];
        else {
            T vl = _query(a, b, k*2+1, l, (l+r)/2);
            T vr = _query(a, b, k*2+2, (l+r)/2, r);
            return max(vl, vr);
        }
    }
    // [a, b)
    T query(int a, int b) {
        return _query(a, b, 0, 0, n);
    }
    // a
    T query(int a) {
        return query(a, a+1);
    }
};

// 1～n に圧縮
vll compress(vll ary) {
    vll tmp;
    tr(it, ary) tmp.push_back(*it);
    sort(all(tmp));
    tmp.erase(unique(all(tmp)), tmp.end());

    tr(it, ary)
        *it = lower_bound(all(tmp), *it) - begin(tmp) + 1;
    return ary;
}


int main() {
    int n;
    cin >> n;

    vll cakes(n);
    rep(i, n) {
        int r, h;
        cin >> r >> h;
        cakes[i] = (ll)r * r * h;
    }

    vll comp = compress(cakes);

    // seg[v] := 体積vが一番上に乗ったケーキたちの最大体積
    SegTree<ll> seg(n+1);
    seg.update(0, 0);
    rep(i, n) {
        // segに既に存在する、
        //  一番上が自分より小さいもの [0, comp[i]) から最大を探す
        double mx = seg.query(0, comp[i]);
        seg.update(comp[i], mx + cakes[i]);
    }

    double ans = seg.query(0, n+1);
    cout << ans * PI << endl;
    
    return 0;
}
```


## まとめ
参加者は3046人。

- A: 2858/2956
- B: 2305/2571
- C: 176/577
- D: 200/855
- E: 15/28

Cが難しい回だった。

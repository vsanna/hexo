title: "Codeforces #339 Div.2 A-D"
date: 2016-01-20 22:44:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Link/Cut Tree
[問題文](http://codeforces.com/contest/614/problem/A)

### 考え方
制約的にA問題としては罠な感じ。
プレテストが弱かったので、大量虐殺が生まれていた。

l, rが64bit整数の限界辺りなので、普通に列挙するとオーバーフローして誤った答えを出してしまうケースがある。
そこで、掛け算をする前にlogを取って、限界を超えないかを先にチェックしておけばよい。

```C++
int main() {
	unsigned long long l,r,k;
    cin >> l >> r >> k;

    vll ans;
    int c = 0;

    long long now = 1;
    rep(i, 70) {
        if(l <= now && now <= r)
            ans.push_back(now);
        double t = log10(now) + log10(k);

        if(t <= 18.0)
            now *= k;
        else
            break;
    }

    if(ans.empty())
        cout << -1 << endl;
    else {
        tr(it, ans)
            cout << *it << " ";
        cout << endl;
    }
	
    return 0;
}
```

## B. Gena's Code
[問題文](http://codeforces.com/contest/614/problem/B)

### 考え方
最初は問題の意味が分からなかった。
結局は全部掛け算した結果を求めるというシンプルな問題だった。
ただし、数が大きいので、数値として扱うとあふれる。

問題の制約上、beautiful numberでないものに、beautiful numberで使われている0を追加してあげれば、簡単に文字列として計算が出来る。

```C++
int main() {
	int n;
    cin >> n;

    vector<string> beautiful ;
    string badstr = "1";

    rep(i,n) {
        string s;
        cin >> s;

        if(s=="0") {
            cout << 0 << endl;
            return 0;
        }

        int one = 0;
        bool bad = false;
        tr(it, s) {
            if(*it == '1')
                one++;
            else if(*it != '0') {
                bad = true;
                break;
            }
        }

        if(bad || one != 1)
            badstr = s;
        else
            beautiful.push_back(s);
    }

    int zeros = 0;
    tr(it, beautiful) {
        tr(c, *it) {
            if(*c=='0')
                zeros++;
        }
    }

    rep(i, zeros)
        badstr += "0";

    cout << badstr << endl;
	
    return 0;
}
```

## C. Peter and Snow Blower
[問題文](http://codeforces.com/contest/614/problem/C)

### 考え方
Pから最も遠い点が描く円から、最も近い点が描く円を引けば答えになる。
Pと各点の距離だけ見れば十分、と思ったが、そんな簡単な訳がなくプレテスト3で落ちる。

点と直線の距離は、垂線の足が最短になる。
というわけで、Pと辺上の点が最短距離になることがある。
丁度サンプルの三角形が上下反転したような形だとこのような状況が発生する。

垂線の足を求めるため、Pから直線に垂線を下ろし、それが線分の範囲内に入っているかを判定した。
交点が求まった後は、通常通り距離の計算を行えば良い。
後で二乗するため、ルートを取らずに距離を計算し誤差を減らしている。

```C++
const static double PI = 4*atan(1.0);
typedef pair<double, double> pdd;

double dist2(pdd A, pdd B) {
    double tx = A.first, x = B.first;
    double ty = A.second, y = B.second;
    return (tx-x)*(tx-x) + (ty-y)*(ty-y);
}

double intersectPoint(pdd P, pdd A, pdd B) {
    if(A > B)
        swap(A, B);

    double a = A.second - B.second;
    double b = -(A.first - B.first);
    double c = -a*A.first - b*A.second;

    // H = P - T*(a,b)
    double T = (a*P.first + b*P.second + c) / (a*a + b*b);
    pdd H = mp( P.first - T*a, P.second - T*b );

    if(A.first <= H.first && H.first <= B.first)
        return dist2(P, H);
    else
        return -1.0;
}

int main() {
    ll n, x, y;
    cin >> n >> x >> y;
    pdd P(x, y);

    double mxdist = -1, midist = numeric_limits<double>::max();
    vector<pdd> coords(n);

    rep(i,n) {
        ll tx, ty;
        cin >> tx >> ty;
        coords[i] = mp(tx, ty);

        // point to point
        double dist = dist2(P, mp(tx,ty));
        chmax(mxdist, dist);
        chmin(midist, dist);
    }
    rep(i, n) {
        // point to segment
        double dist = intersectPoint(P, coords[i%n], coords[(i+1)%n]);
        if(dist > 0.0)
            chmin(midist, dist);
    }

    double diff = mxdist - midist;
    cout << diff * PI << endl;
	
    return 0;
}
```

## D. Skills
[問題文](http://codeforces.com/contest/614/problem/D)

### 考え方
最大の人数と、最小の値を求めれば良いが、値が大きいので全探索と二分探索を組み合わせる。
最大の人数は$n$以下なので、こちらを全探索し、
最小の値は[1, A]を二分探索すればよい。
オーダーは$O(nlogA)$。

探索しているそれぞれの値がコストの条件を満たしているかの判定は、

- 探索した最大の人数にするのに必要なコスト
- 探索した最小の値にするのに必要なコスト

をそれぞれ累積和を使って保持し利用する。
「最大の人数」の所要コストは、大きい値を持っている順に計算すれば簡単に求まる。
「最小の値」の所要コストは、小さい値順に累積和を取り、人数×値から累積和を引いて求める事ができる。

```C++

int main() {
	int n, A, cf, cm;
    ll m;
    cin >> n >> A >> cf >> cm >> m;

    vector<pii> ary(n);
    rep(i,n) {
        cin >> ary[i].first;
        ary[i].second = i;
    }
    sort(all(ary));

    vll accum_sum(n+1, 0);
    rep(i, n)
        accum_sum[i+1] = ary[i].first + accum_sum[i];

    reverse(all(ary));
    vll need_max(n+1, 0);
    rep(i, n)
        need_max[i+1] = A - ary[i].first + need_max[i];
    reverse(all(ary));

    ll max_score = 0;
    int min_num=0, max_num=0, min_val=0;
    rep(alpha, n+1) {
        ll rem = (ll)m - need_max[alpha];
        if(rem < 0)
            break;

        int lb = 0;  // minimum possible
        int ub = A+1;  // maximum impossible
        while(ub - lb > 1) {
            int beta = (lb + ub) / 2;
            // betaより小さい要素の個数
            int ind = distance(begin(ary), lower_bound(begin(ary), begin(ary) + (n-alpha), mp(beta, -1)));
            ll cost = (ll) ind * beta - accum_sum[ind];

            if(rem >= cost) {  // possible
                if(chmax(max_score, (ll)alpha*cf + (ll)beta*cm))
                    min_num = ind, min_val = beta, max_num = alpha;
                lb = beta;
            }
            else
                ub = beta;
        }
    }

    rep(i, min_num)
        ary[i].first = min_val;
    rep(i, max_num)
        ary[n-i-1].first = A;

    vi ans(n);
    rep(i, n)
        ans[ary[i].second] = ary[i].first;

    cout << max_score << endl;
    tr(it, ans)
        cout << *it << " ";
    cout << endl;

    return 0;
}
```


## まとめ
参加者は3752人。

- A: 913/3664
- B: 1571/2851
- C: 310/1445
- D: 41/178
- E: 2/29

Aが恐ろしいHack数で、+10以上Hackした人が大量にいた。
大混乱のセットだったが、問題としては良かったと思うので普通に解きたかった。


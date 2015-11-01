title: "Ari-practice solve 10"
date: 2015-11-01 22:15:31
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (10)
中級編。

## POJ3258 - River Hopscotch
答えを決め打ちしてにぶたん。
間隔を事前に計算しておき、条件を満たさない場合は次の間隔に今の間隔を加えると楽に計算できた。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int L, N, M;
    cin >> L >> N >> M;

    vi stones;
    stones.push_back(0);
    rep(i, N) {
        int t;
        cin >> t;
        stones.push_back(t);
    }
    stones.push_back(L);

    sort(all(stones));

    vi dists(stones.size()-1, 0);
    REP(i, 1, stones.size())
        dists[i-1] = stones[i] - stones[i-1];

    int lb = 0;  // maximum possible
    int ub = L + 10;  // minimum impossible
    while(ub - lb > 1) {
        int med = (ub+lb)/2;

        vi cd;
        copy(dists.begin(), dists.end(), back_inserter(cd));
        int mcount = 0;
        rep(i, cd.size()) {
            if(cd[i] < med) {
                mcount++;
                if(i < cd.size()-1)
                    cd[i+1] += cd[i];
            }
        }

        if(mcount <= M)
            lb = med;
        else
            ub = med;
    }

    cout << lb << endl;

    return 0;
}
```

## POJ3273 - Monthly Expense
↑と大体同じ。書くだけ。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    vi cost(N, 0);
    rep(i, N) cin >> cost[i];

    int lb = 0;  // maximum impossible
    int ub = (int)(1e9) + 100;  // minimum possible

    while(ub - lb > 1) {
        int med = ((ll)lb + ub) / 2;

        // greedy
        int nowCost = 0;
        int count = 0;
        bool possible = true;
        rep(i, N) {
            if(med < cost[i]) {
                possible = false;
                break;
            }
            if(med < nowCost + cost[i]) {
                count++;
                nowCost = 0;
            }
            nowCost += cost[i];
        }
        count++;

        if(!possible) {
            lb = med;
            continue;
        }

        if(count <= M)
            ub = med;
        else
            lb = med;
    }

    cout << ub << endl;

    return 0;
}
```

## POJ3104 - Drying
「自然乾燥に加えてk乾燥させられる」と勘違いしてWA。
修正したら今度はコーナーケースに引っかかった。
考え方自体は↑と同じで特にひねりはない。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;
    vi a(N);
    rep(i,N) cin >> a[i];
    int k;
    cin >> k;

    int lb = 1;  // maximum impossible
    int ub = *max_element(all(a));  // minimum possible

    if(k == 1) {
        cout << ub << endl;
        return 0;
    }

    while(ub - lb > 1) {
        int med = ((ll)ub + lb) / 2;
        ll kcount = 0;
        rep(i, N) {
            if( med < a[i] ) {
                int over = a[i] - med;
                kcount += ceil((double)over / (k-1));
            }
        }

        if(kcount <= med)
            ub = med;
        else
            lb = med;
    }

    cout << ub << endl;

    return 0;
}
```


## POJ3045 - Cow Acrobats
[ここをみた](https://wiki.mma.club.uec.ac.jp/iz/%E7%AB%B6%E6%8A%80%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0/PKU%203045%20Cow%20Acrobats)
リスクをにぶたんしても、結局牛の並び替えがわからんなー、と思っていたらまさかの貪欲。
数式変形力が必要だった。

```C++

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    vector< pair<ll, ll> > ws_w(N);
    rep(i, N) {
        ll w, s;
        cin >> w >> s;
        ws_w[i] = mp(w+s, w);
    }

    sort(all(ws_w));

    ll res = -1ll * (1ll << 60);
    ll sum = 0;
    rep(i, N) {
        ll s = ws_w[i].first - ws_w[i].second;
        ll w = ws_w[i].second;
        res = max(res, sum-s);
        sum += w;
    }

    cout << res << endl;

    return 0;
}
```

## POJ2976 - Dropping tests
基本的に蟻本の練習問題と同じ。
doubleでにぶたんしないといけないのは注意。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n,k;

    while(cin >> n >> k, (n|k)) {
        vector<pii> ab(n);
        rep(i,n) cin >> ab[i].first;
        rep(i,n) cin >> ab[i].second;

        double lb = 0;  // maximum possible
        double ub = 100;  // minimum impossible

       rep(_loop, 100) {
            double med = (lb + ub) / 2;
            priority_queue<double> score;
            rep(i,n)
                score.push( 100.0 * ab[i].first - med * ab[i].second );
            double judge = 0;
            rep(i,n-k) {
                judge += score.top();
                score.pop();
            }

            if(judge >= 0.0)
                lb = med;
            else
                ub = med;
        }

        cout << (int)round(lb) << endl;
    }

    return 0;
}
```

## POJ3111 - K Best
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/08/23/175658)
アルゴリズム自体は今までと変わらないが、やたら時間制限が厳しくてチューンアップ大会に。
なぜか答えの出力部分がこれだと通らなくて

```C++
for(int i=n-1; i>=n-k; --i) {
    if(i != n-1)
        printf(" ");
    printf("%d", score[i].second);
}
printf("\n");
```

これだと通る。

```C++
    for(int i = 0; i < k; ++i)
        printf("%d%c", y[n-i-1].second, i == k - 1 ? '\n': ' ');
```

printfは分割すると異常に遅くなるのだろうか。
C言語力が足りない……。

それにしても、こういう本質的でないところに罠がある問題は、練習向きではないので辞めていただきたい……。

```C++
int n, k;
const int MAX_N = (int)(1e5) + 10;
vector<pii> vw;
vector< pair<double, int> > score;

vi res;

bool check(double med) {
    priority_queue< pair<double, int> > score;
    rep(i, n)
        score.push( mp( vw[i].first - med * vw[i].second, i + 1 ) );

    double keep_score = 0;
    rep(i, k) {
        keep_score += score.top().first;
        res[i] = score.top().second;
        score.pop();
    }

    return keep_score >= 0.0;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    scanf("%d%d", &n, &k);
    vw.resize(n);
    score.resize(n);
    res.resize(k);
    rep(i, n)
        scanf("%d%d", &vw[i].first, &vw[i].second);

    double lb = 0;   // maximum possible
    double ub = (int)(1e6) + 100;  // minimum impossible

    rep(_loop, 50) {
        double med = (ub + lb) / 2;
        if(check(med))
            lb = med;
        else
            ub = med;
    }

    check(lb);

    rep(i,k)
        printf("%d%c", res[i], i == k-1 ? '\n': ' ');

    return 0;
}
```


## POJ3579 - Median
なかなか面白い問題。
2重のにぶたんで中央値を求めるという、少しトリッキーな方法で計算量を落とす。
気付いた時はスッキリした。
cin, coutを使うとTLEしたので少々イラッとさせられたが、単純に書き換えれば通った。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    while(scanf("%d", &N) != EOF) {
        vi nums(N);
        rep(i, N) scanf("%d", &nums[i]);
        sort(all(nums));

        int lb = 0;
        int ub = (int) (1e9) + 100;

        while (ub - lb > 1) {
            int med = ((ll) lb + ub) / 2;

            ll upper_num = 0;
            rep(i, N) {
                upper_num += nums.end() -
                        lower_bound(nums.begin() + i + 1, nums.end(), nums[i] + med);
            }

            ll diffnum = ((ll)N * (N-1))/2;
            if (upper_num >= diffnum/2 + 1)
                lb = med;
            else
                ub = med;
        }
        printf("%d\n", lb);
    }

    return 0;
}
```


## POJ3685 - Matrix
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/08/25/103419)
方針が浮かばなかった。
jに定数を代入してみると、なんとiに対し単調増加する数式になる。
そのため、にぶたんを2重で行い、基準値以上の値を持つ個数をMと比較する。
にぶたんの条件がややこしく、発想としては単純な割に書き下すのが難しかった。

```C++
ll N, M;

ll f(ll i, ll j) {
    return i*i + (ll)1e5 * i + j*j - (ll)1e5 * j + i*j;
}

bool check(ll target) {
    ll nums = 0;
    REP(j, 1, N+1) {  // jを固定
        int lb = 0;
        int ub = N + 1;
        while(ub - lb > 1) {  // iをにぶたん
            int med = (ub + lb) / 2;
            if(f(med, j) < target)  // 基準より小さい -> 上げる
                lb = med;
            else
                ub = med;
        }
        nums += lb;
    }
    return nums < M;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int __test;
    cin >> __test;
    while(__test--) {
        cin >> N >> M;

        ll lb = -1ll * (ll)(1e11);
        ll ub = -lb;

        while(ub - lb > 1 ) {
            ll target = (ub + lb) / 2;   // 基準のy座標
            if(check(target))  // 数が足りない -> 基準をゆるくする
                lb = target;
            else
                ub = target;
        }
        cout << lb << endl;
    }

    return 0;
}
```

## POJ2010 - Moo University - Financial Aid
2度目の登場。
どれがmedianになるかをにぶたんで決める。
priority_queueを使った解法より楽ちん。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, C, F;
    cin >> N >> C >> F;

    vector<pii> calves(C);
    rep(i, C)
        cin >> calves[i].first >> calves[i].second;

    sort(all(calves));

    int half = (N-1) / 2;
    int result = -1;

    int lb = 0;
    int ub = C-1;
    while(ub - lb > 1) {
        int med = (lb + ub) / 2;

        priority_queue< pii, vector<pii>, greater<pii> > lower;
        rep(i, med)
            lower.push(mp(calves[i].second, calves[i].first));
        priority_queue< pii, vector<pii>, greater<pii> >  upper;
        REP(i, med+1, C)
            upper.push(mp(calves[i].second, calves[i].first));

        // 数が足りない
        if(lower.size() < half) {
            lb = med;
            continue;
        }
        if( upper.size() < half) {
            ub = med;
            continue;
        }

        // コスト計算
        ll cost = calves[med].second;
        rep(i, half) {
            pii t1 = lower.top(); lower.pop();
            pii t2 = upper.top(); upper.pop();
            cost += ((ll)t1.first + t2.first);
        }

        if(cost <= F) {
            lb = med;
            result = med;
        }
        else
            ub = med;
    }

    if(result == -1)
        cout << result << endl;
    else
        cout << calves[result].first << endl;

    return 0;
}
```


## POJ3662 - Telephone Lines
グラフ+にぶたん。[ここを見た](http://pushl.hatenablog.com/entry/2014/01/21/034235)
根幹のアルゴリズムとしては、にぶたんで答えを先に決め、1->Nの経路で条件を満たすものがあるかを求める。
経路探索での工夫として、コストを0か1にしてしまうと楽になる。
先に決めた答えを上回る場合は1、それ以外は0にすると、単純なダイクストラで最短経路を求めると必要なKの数が求まる。

```C++
int N, P, K;
struct Edge {int to, length; };
vector< vector<Edge> > edges;

int check(int med) {
    priority_queue< pii, vector<pii>, greater<pii> > que;
    vector<bool> visited(N, false);
    que.push(mp(0, 0));

    while(!que.empty()) {
        pii p = que.top(); que.pop();
        int c = p.first;
        int v = p.second;

        if(visited[v])
            continue;
        if(v == N-1)
            return c;

        visited[v] = true;

        tr(it, edges[v]) {
            if(it->length <= med)
                que.push(mp(c, it->to));
            else
                que.push(mp(c+1, it->to));
        }
    }

    return -1;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    cin >> N >> P >> K;

    edges.resize(N);
    rep(i, P) {
        int a, b, l;
        cin >> a >> b >> l;
        --a, --b;
        edges[a].push_back(Edge{b, l});
        edges[b].push_back(Edge{a, l});
    }

    const int INF = (int)(1e6) + 100;
    int lb = 0;  // maximum impossible
    int ub = INF; // minimum possible
    rep(_t, 20) {
        int med = (lb + ub) / 2;
        int t = check(med);
        if(0 <= t && t <= K)
            ub = med;
        else
            lb = med;
    }

    if(ub >= INF)
        cout << -1 << endl;
    else
        cout << ub << endl;

    return 0;
}
```

## POJ1759 - Garland
[ここを見た](http://poj.org/problem?id=1759)
考え方は分かったが、しょうもないバグで他サイトを見るはめになった。
今回は2つ目の値をにぶたんし、求めたいN番目の値を計算するパターン。

```C++
int N;
double A;
bool check(double preprev, double prev, double& tmp) {
    REP(i, 3, N+1) {  // H3 -> hN
        double hi = 2.0 * prev + 2.0 - preprev;
        if(hi < 0)
            return false;
        preprev = prev;
        prev = hi;
    }
    tmp = prev;
    return true;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(2);
    ios_base::sync_with_stdio(false);

    cerr.setf(ios::fixed, ios::floatfield);
    cerr.precision(8);

    cin >> N >> A;

    double b_lb = 0.0;  // maximum impossible
    double b_ub = 1e4 + 10.0;  // minimum possible

    double ans = numeric_limits<double>::max();
    rep(_t, 1000) {
        double h2 = (b_ub + b_lb) / 2;
        double tmp = 0.0;
        if(check(A, h2, tmp)) {
            b_ub = h2;
            ans = min(ans, tmp);
        }
        else
            b_lb = h2;
    }

    cout << ans << endl;

    return 0;
}
```

## POJ3484 - Showstopper
「欠損データがどこかにある」ということを活かし、「med以下の個数が奇数かどうか」を判定することでにぶたんができる。

getlineで書いていたがおかしいらしく、[ここを見て](http://d.hatena.ne.jp/komiyam/20120207/1328545633)入力部分を頂いた。
POJだとダメなインプットが分からず、流石に時間の無駄なので原因究明は諦めた。

```C++
void solve(vector<ll>& Xs, vector<ll>& Ys, vector<ll>& Zs){
    int N = Xs.size();

    // 欠損しているか最初にチェック
    {
        ll num = 0;
        rep(i, N)
            num += (Ys[i] - Xs[i]) / Zs[i] + 1;
        if(num % 2 == 0) {
            cout << "no corruption" << endl;
            return;
        }
    }

    // med以下の数について個数をカウント。
    // 奇数個なら、それ以下に欠損している数がある
    ll lb = 0;
    ll ub = (ll)(1e10);  // odd starts from this num
    while(ub - lb > 1) {
        ll med = (lb + ub) / 2;

        ll num = 0;
        rep(i, N) {
            if(med < Xs[i])
                continue;
            num += (min((ll)Ys[i], med) - Xs[i]) / Zs[i] + 1;
        }

        if(num % 2 == 0)
            lb = med;
        else
            ub = med;
    }

    // 答えが決まったら、個数をカウント
    ll res = ub;
    int count = 0;
    rep(i, N) {
        if(Xs[i] <= res && res <= Ys[i] && (res-Xs[i]) % Zs[i] == 0)
            count++;
    }
    cout << res << " " << count << endl;
}

int main(){
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    vector<ll> Xs, Ys, Zs;
    while(true){
        {  // input
            Xs.clear();
            Ys.clear();
            Zs.clear();

            int f = 0;
            char line[1025];
            while ((f = ((gets(line) != NULL))) && strlen(line) > 2) {
                uint X, Y, Z;
                sscanf(line, "%u %u %u", &X, &Y, &Z);

                Xs.push_back(X);
                Ys.push_back(Y);
                Zs.push_back(Z);
            }

            if(!f && Xs.empty())
                break;

            if(Xs.empty())
                continue;
        }

        solve(Xs, Ys, Zs);
    }

    return 0;
}
```


## まとめ
ひたすらに二分探索をした。
振り返ってみると、なんと12問もあった。
色々な問題で使えることが分かったので、解法の幅が広がった感じがする。
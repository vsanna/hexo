title: "Ari-practice solve #7"
date: 2015-09-11 00:35:11
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (7)
データ構造はアルゴリズム的には簡単な分、実装が重い……。

## POJ3614 - Sunscreen
[ここを見た](http://d.hatena.ne.jp/sune2/20120719/1342658409)
まず、ボトルをSPFが低い順に並べる。
そのボトルで上手くいく牛の中で、最大値が一番低い牛に貪欲に使ってゆく。
なるべく条件が厳しい牛に使ってゆくため、貪欲法で上手くいく。
計算量の見積もりが難しい問題だが、おおよそ$O(LC)$くらいには収まるように思う。
dinicなるものを使うと解けるらしいが、最大流は良く分からない……。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int C, L;
    cin >> C >> L;

    vector<pii> mi_mx_SPF(C);
    rep(i, C) cin >> mi_mx_SPF[i].first >> mi_mx_SPF[i].second;
    sort(all(mi_mx_SPF));

    priority_queue<pii, vector<pii>, greater<pii> > SPF_cover;
    rep(i, L) {
        int SPF, cover;
        cin >> SPF >> cover;
        SPF_cover.push(mp(SPF, cover));
    }

    int res = 0;
    while(!SPF_cover.empty()) {
        pii spf = SPF_cover.top(); SPF_cover.pop();

        // 範囲内の牛を、最大許容SPFが小さい順になるよう入れる
        priority_queue<pii, vector<pii>, greater<pii> > que;  // mx_mi
        rep(i, mi_mx_SPF.size())
            if(mi_mx_SPF[i].first <= spf.first && spf.first <= mi_mx_SPF[i].second)
                que.push(mp(mi_mx_SPF[i].second, i));

        // 条件に合う牛がいた
        if(!que.empty()) {
            pii t = que.top(); que.pop();
            mi_mx_SPF.erase(mi_mx_SPF.begin() + t.second);
            res++;

            // 残っていたら戻す
            if(spf.second > 1)
                SPF_cover.push(mp(spf.first, spf.second-1));
        }
    }

    cout << res << endl;

    return 0;
}
```

## POJ2010 - Moo University - Financial Aid
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/08/08/232903)
上手いこと貪欲しないとTLEする。
思いつかん…。

```C++
int N, C, F, n;
const int MAX_C = 100005;
pii csat_aid[MAX_C];
ll upper[MAX_C], lower[MAX_C];
const ll INF = 1ll<<60;


// iより小さいのから、コストが最小になるようにK個選ぶ
void calc(ll* array) {
    ll res = 0ll;
    priority_queue<int> que;

    // 先頭n個はとりあえず入れる
    rep(i,n) {
        que.push(csat_aid[i].second);
        res += csat_aid[i].second;
        // 使えないのでINFを入れとく
        array[i] = INF;
    }

    REP(i, n, C) {
        array[i] = res;
        // 入れ替えてより良くなるなら
        if(que.top() > csat_aid[i].second) {
            int t = que.top(); que.pop();
            res = res - t + csat_aid[i].second;
            que.push(csat_aid[i].second);
        }
    }
}


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    cin >> N >> C >> F;

    rep(i, C)
        cin >> csat_aid[i].first >> csat_aid[i].second;
    sort(csat_aid, csat_aid+C);

    n = N/2;

    calc(lower);
    reverse(csat_aid, csat_aid+C);
    calc(upper);
    reverse(upper, upper+C);
    reverse(csat_aid, csat_aid+C);

    int mxmed = -1;
    rep(i, C) {
        if(lower[i] + upper[i] + csat_aid[i].second <= F) {
            mxmed = max(mxmed, csat_aid[i].first);
        }
    }

    cout << mxmed << endl;

    return 0;
}
````


## POJ2236 - Wireless Network
どう実装したものか少し悩んだが、UFを使うと分かっていたのでなんとかなった。
UF部分のコードは省略。


```C++

int twice(int t) { return t*t; }

int d2range(pii p1, pii p2) {
    return twice(p1.first - p2.first) + twice(p1.second - p2.second);
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, d;
    cin >> N >> d;
    UnionFind uf(N);

    vector<pii> points(N);
    rep(i, N)
        cin >> points[i].first >> points[i].second;

    vector<bool> fixed(N, false);

    char c;
    while(cin >> c) {
        if(c=='O') {
            int fix = 0;
            cin >> fix;
            fix--;
            if( ! fixed[fix]) {
                fixed[fix] = true;

                // chain
                rep(i,N) {
                    if( d2range(points[i], points[fix]) <= d*d && fixed[i])
                        uf.unite(i,fix);
                }
            }
        }
        else {
            int f, s;
            cin >> f >> s;
            f--; s--;
            if( uf.same(f, s) && fixed[f] && fixed[s] )
                cout << "SUCCESS" << endl;
            else
                cout << "FAIL" << endl;
        }
    }

    return 0;
}
```



## POJ1703 - Find them, Catch them
蟻本と同じ……というヒントを見てなんとか解けた。
こんな使い方も出来るのかー、と感心。
cinを使うとTLEした。

```C++

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int T;
    //cin >> T;
    scanf("%d", &T);

    while(T--) {
        int N,M;
        //cin >> N >> M;
        scanf("%d%d", &N, &M);

        UnionFind uf;
        uf.init(N*2);

        while(M--) {
            char c[16];
            int a, b;
            //cin >> c >> a >> b;
            scanf("%s%d%d", c, &a, &b);

            a--; b--;

            if (c[0] == 'A') {
                if (uf.same(a, b))
                    cout << "In the same gang." << endl;
                else if (uf.same(a, b + N))
                    cout << "In different gangs." << endl;
                else
                    cout << "Not sure yet." << endl;
            }
            else {
                uf.unite(a, b + N);
                uf.unite(a + N, b);
            }
        }
    }

    return 0;
}
```

## まとめ
同じデータ構造を使う問題なので、似たり寄ったりだった。
その中でも色々パターンがあるので、やっておいて損はなさそう。
特にUnionFindは思ったより幅広く使われていそう。

title: "Ari-practice solve #8"
date: 2015-09-24 21:58:11
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (8)
ついにグラフ。
グラフ単品だと簡単な問題が多かった。

## AOJ0189 - Convenient Location
ワーシャルフロイドするだけ。

```C++
const int MAX_N = 11;
int d[MAX_N][MAX_N];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    while(cin >> n, n) {
        rep(i,MAX_N) {
            fill(d[i], d[i] + MAX_N, INF);
            d[i][i] = 0;
        }

        rep(i,n) {
            int a, b, c;
            cin >> a >> b >> c;
            d[a][b] = d[b][a] = c;
        }

        rep(k, MAX_N) {
            rep(i, MAX_N) {
                rep(j, MAX_N) {
                    d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
                }
            }
        }

        pair<ll, int> res = mp(INF, 1);
        rep(i,MAX_N) {
            ll total = 0;
            rep(j,MAX_N)
                if(d[i][j] == INF)
                    break;
                else
                    total += d[i][j];
            if(total != 0 && total < res.first)
                res = mp(total, i);
        }

        cout << res.second << " " << res.first << endl;
    }


    return 0;
}
```

## POJ2139 - Six Degrees of Cowvin Bacon
こっちもワーシャルフロイドするだけ。

```C++
const int MAX_N = 301;
int d[MAX_N][MAX_N];
int tmpM[10010];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    rep(i,MAX_N) {
        fill(d[i], d[i] + MAX_N, INF);
        d[i][i] = 0;
    }

    int N, M;
    cin >> N >> M;

    rep(i,M) {
        int len;
        cin >> len;
        rep(j,len) cin >> tmpM[j];

        rep(j,len) {
            int from = tmpM[j] - 1;
            REP(k, j+1, len) {
                int to = tmpM[k] - 1;
                d[from][to] = d[to][from] = 1;
            }
        }
    }

    // WF
    rep(k,N) {
        rep(i,N) {
            rep(j,N) {
                d[i][j] = min(d[i][j], d[i][k] + d[k][j]);
            }
        }
    }

    int res = INF;
    rep(i,N) {
        int total = 0;
        rep(j,N) {
            if(i == j) continue;
            total += d[i][j];
        }
        res = min(res, total);
    }

    cout << (res*100) / (N-1) << endl;

    return 0;
}
```

## POJ3259 - Wormholes
負の閉路が出来るかどうかをチェックするだけ。
-1でも負になるなら、そこを回り続ければいくらでもマイナスにできるため。

```C++
struct edge {int from,to,cost;};

bool bellman(const int N, vector< vector<edge> >& graph) {
    vi dist(N, 0);
    int E = graph.size();

    rep(i, N) {
        rep(j, E) {
            vector<edge>& e = graph[j];
            tr(it, e){
                if(dist[it->to] > dist[it->from] + it->cost) {
                    dist[it->to] = dist[it->from] + it->cost;
                    if(i == N-1)
                        return true;
                }
            }
        }
    }
    return false;
}


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int F;
    cin >> F;
    rep(_prob, F) {
        int N, M, W;
        cin >> N >> M >> W;

        vector< vector<edge> > graph(N);

        rep(i, M){
            int S, E, T;
            cin >> S >> E >> T;
            --S, --E;
            graph[S].push_back(edge{S, E, T});
            graph[E].push_back(edge{E, S, T});
        }
        rep(i, W){
            int S, E, T;
            cin >> S >> E >> T;
            --S, --E;
            graph[S].push_back(edge{S, E, -T});
        }

        if(bellman(N, graph))
            cout << "YES" << endl;
        else
            cout << "NO" << endl;
    }

    return 0;
}
```


## POJ3268 - Silver Cow Party
辺の向きを逆にすることで、各点->会場を1回のダイクストラで表すことが出来る。
結果、2回ダイクストラを行うだけで求められる。

```C++
struct edge{ int to, cost;};

void dijkstra(int start, vector< vector<edge> > &graphs, vi& res) {
    priority_queue<pii, vector<pii>, greater<pii> > que;
    int N = res.size();
    vi d(N, INF);
    d[start] = 0;
    que.push(mp(0, start));

    while(!que.empty()){
        pii p = que.top(); que.pop();
        int v = p.second;
        if(d[v] < p.first)
            continue;

        tr(it, graphs[v]) {
            if(d[it->to] > d[v] + it->cost) {
                d[it->to] = d[v] + it->cost;
                que.push(mp(d[it->to], it->to));
            }
        }
    }

    rep(i, N)
        res[i] += d[i];
}


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M, X;
    cin >> N >> M >> X;
    --X;

    vector< vector<edge> > graphs(N);
    vector< vector<edge> > graphs2(N);

    rep(i,M) {
        int A, B, T;
        cin >> A >> B >> T;
        A--, B--;
        graphs[A].push_back(edge{B, T});
        graphs2[B].push_back(edge{A, T});
    }

    vi res(N, 0);
    dijkstra(X, graphs, res);
    dijkstra(X, graphs2, res);

    cout << *max_element(all(res)) << endl;

    return 0;
}
```


## AOJ2249 - Road Construction
元のプランと最短距離が同じなので、最短距離で使われる道の中ででコストが最小になる組み合わせを選べば良い。
そのため、ダイクストラに少し工夫をしたうえで、経路復元をすれば求まる。

```C++
struct Edge {int to, dist, cost;};
typedef vector< vector<Edge> > Graph;
 
int N;
void dijkstra(int start, Graph& graph) {
    priority_queue<pii, vector<pii>, greater<pii> > que;
    vi d(N, INF);
    d[start] = 0;
    que.push(mp(0, start));
 
    vi prev(N, -1);
    vi pc(N, -1);
 
    while(!que.empty()) {
        pii p = que.top(); que.pop();
        int v = p.second;
        if(d[v] > p.first) continue;
        tr(it, graph[v]) {
            if(
                (d[it->to] > d[v] + it->dist) ||
                (d[it->to] == d[v] + it->dist && pc[it->to] > it->cost)
                    )
            {
                d[it->to] = d[v] + it->dist;
                que.push(mp(d[it->to], it->to));
                prev[it->to] = v;
                pc[it->to] = it->cost;
            }
        }
    }

    cout << accumulate(pc.begin() + 1, pc.end(), 0) << endl;
 
}
 
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int M;
    while(cin >> N >> M, (N|M)) {
        Graph graph(N);
        rep(i, M) {
            int u, v, d, c;
            cin >> u >> v >> d >> c;
            --u, --v;
            graph[u].push_back(Edge{v, d, c});
            graph[v].push_back(Edge{u, d, c});
        }

        dijkstra(0, graph);
    }

    return 0;
}
```


## AOJ2200 - Mr. Rito Post Office
突然のグラフ+DP問題。難しい。
[ここを見た。](http://zaburoapp.blog.fc2.com/blog-entry-20.html)
船の位置に着目すると、移動が下記の2パターンに絞られる。
結果、船の位置でDPすれば上手くいく。

- 陸路のみ
- 陸路 -> 海路 -> 陸路

なお、海路 -> 陸路 -> 海路 とすると、陸路で同じ島に戻ってこないと船が使えないので無駄。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);
 
    int N, M;
    while(cin >> N >> M, (N|M)) {
        vector< vi > dl(N, vi(N, INF));
        vector< vi > ds(N, vi(N, INF));
        rep(i, N) {
            dl[i][i] = ds[i][i] = 0;
        }
 
        rep(i, M) {
            int x, y, t;
            char sl;
            cin >> x >> y >> t >> sl;
            --x, --y;
            if(sl == 'L')
                dl[x][y] = dl[y][x] = t;
            else
                ds[x][y] = ds[y][x] = t;
        }
 
        // WF
        rep(k, N) {
            rep(i, N) {
                rep(j, N) {
                    dl[i][j] = min(dl[i][j], dl[i][k] + dl[k][j]);
                    ds[i][j] = min(ds[i][j], ds[i][k] + ds[k][j]);
                }
            }
        }
 
        int R;
        cin >> R;
        vi z(R);
        rep(i, R) {
            int tz;
            cin >> tz;
            z[i] = (--tz);
        }
 
        // dp[i][j] := z[i]まで移動し、船がjにある時の最小累計距離
        vector<vi> dp(R, vi(N,INF));
        dp[0][z[0]] = 0;
 
        REP(i, 1, R) {
            rep(j, N) {
                // 陸路のみ
                // もともと船がjにある場合のみ
                if(dp[i-1][j] != INF) {
                    dp[i][j] = min(dp[i][j],
                                   dp[i - 1][j] + dl[z[i - 1]][z[i]]);
                }
                // 陸路 -> 海路 -> 陸路
                // 海路でjに来る場合
                rep(k, N) {
                    if(dp[i-1][k] != INF) {
                        dp[i][j] = min(dp[i][j],
                                       dp[i - 1][k] + dl[z[i - 1]][k] + ds[k][j] + dl[j][z[i]]
                        );
                    }
                }
            }
        }
 
        cout << *min_element(all(dp[R-1])) << endl;
    }
 
    return 0;
}
```

## POJ1258 - Agri-Net
ただプリムするだけ。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    while(cin >> N) {
        vector<vi> cost(N, vi(N, INF));

        rep(i, N)
            rep(j, N)
                cin >> cost[i][j];

        vi mincost(N, INF);
        vector<bool> used(N, false);

        mincost[0] = 0;
        int res = 0;

        while(true) {
            int v = -1;
            rep(u, N) {
                if(!used[u] && (v == -1 || mincost[u] < mincost[v]))
                    v = u;
            }

            if(v == -1)
                break;

            used[v] = true;
            res += mincost[v];

            rep(u, N)
                mincost[u] = min( mincost[u], cost[v][u] );
        }
        cout << res << endl;
    }

    return 0;
}
```


## POJ2377 - Bad Cowtractors
最大全域木。
ただコストを負にしただけのクラスカル。

```C++
struct edge {int u, v, cost;};
bool comp(const edge& e1, const edge& e2) {
    return e1.cost < e2.cost;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    vector<edge> edges;

    rep(i, M) {
        int a, b, c;
        cin >> a >> b >> c;
        edges.push_back( edge{(--a),(--b),-c} );
    }
    sort(all(edges), comp);

    UnionFind uf(N);
    int res = 0;

    tr(it, edges){
        if(!uf.same(it->u, it->v)) {
//            cerr << it->u << " " << it->v << " " << it->cost <<endl;
            uf.unite(it->u, it->v);
            res += it->cost;
        }
    }

    rep(i, N) {
        if(!uf.same(0, i)){
            cout << -1 << endl;
            return 0;
        }
    }

    cout << -res << endl;

    return 0;
}
```

## AOJ2224 - Save your cats
面白い問題。
フェンスの繋がりを木と捉えて、閉路を消す最小コストを求める。
全域木を構成して**いない**辺のコストが最小になればいいので、最大全域木を作ると上手くいくことが分かる。

```C++
struct edge {int from, to; double cost;};
bool comp(const edge& e1, const edge& e2) {
    return e1.cost < e2.cost;
}
ll twice(int t) {return t*t;}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    vector< pii > piles(N);
    rep(i, N)
        cin >> piles[i].first >> piles[i].second;

    vector<edge> edges(M);
    double totalCost = 0.0;
    rep(i, M) {
        int p, q;
        cin >> p >> q;
        --p, --q;
        double cost = sqrt( twice(piles[p].first - piles[q].first) + twice(piles[p].second - piles[q].second) );
        edges[i] = edge{ p, q, -cost };
        totalCost += cost;
    }

    // kruscal
    sort(all(edges), comp);
    UnionFind uf(N);

    double res = 0;

    tr(it, edges) {
        if(!uf.same(it->from, it->to)) {
            uf.unite(it->from, it->to);
            res += it->cost;
        }
    }

    cout << totalCost - (-res) << endl;

    return 0;
}
```


## POJ2395 - Out of Hay
問題の内容から、最小全域木を作るだけだと分かる。
ここに来て簡単な問題。
これ最初の方がいいような。

```C++

struct edge {
    int from, to, cost;
};
bool comp(const edge &e1, const edge &e2) {
    return e1.cost < e2.cost;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    vector<edge> edges(M);
    rep(i, M) {
        int a, b, c;
        cin >> a >> b >> c;
        --a, --b;
        edges[i] = edge{a, b, c};
    }
    sort(all(edges), comp);

    // kruscal
    UnionFind uf(N);
    int res = 0;

    tr(it, edges) {
        if (!uf.same(it->from, it->to)) {
            uf.unite(it->from, it->to);
            res = max(res, it->cost);
        }
    }

    cout << res << endl;

    return 0;
}
```


## まとめ
最小全域木、面白い。
使うだけの問題が多かったが、様々な問題で絡んでくるはずなので、練習の機会には事欠かなさそう。

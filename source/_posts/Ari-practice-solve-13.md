title: "Ari-practice solve 13"
date: 2015-11-29 11:14:30
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (13)
ビットDPと行列累乗。
DPを工夫する系で、割とよく見かける。

## POJ2441 - Arrange the Bulls
普通にビットDP。
制限時間が4sだったので普通に通った。

```C++
const int MAX_M = 20;
int dp[2][1<<MAX_M];

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M;
    cin >> N >> M;

    vector<vi> barns(N);
    rep(i, N) {
        int P;
        cin >> P;
        rep(j, P) {
            int t;
            cin >> t;
            --t;
            barns[i].push_back(t);
        }
    }

    memset(dp, 0, sizeof(dp));
    dp[0][0] = 1;

    int cur = 0, next = 1;
    rep(cow, N) {
        memset(dp[next], 0, sizeof(dp[next]));
        rep(i, 1<<M) {
            if(dp[cur][i] == 0)
                continue;
            tr(p, barns[cow]) {
                if( (i & (1 << *p)) == 0 ) {  // 未使用
                    dp[next][i | (1<<*p)] += dp[cur][i];
                }
            }
        }

        swap(cur, next);
    }

    int res = 0;
    rep(i, 1<<M) {
        res += dp[cur][i];
    }
    cout << res << endl;

    return 0;
}
```


## POJ3254 - Corn Fields
前の行を保持して、列単位でビットDP。
愚直に実装すると、$M \times (1<<N) \times (1<<N) \times N$になり、おおよそ$10^9$でTLEしそう。
一番後ろのNは、

- infertileの箇所を利用していないか？
- 隣接して立っているビットはあるか？
- 上のマスで既に利用していないか？

のチェックを行うためのもの。
これらをビット演算で頑張って$O(1)$で行えば、$10^8$程度に抑えられて間に合う。

```C++
const int MOD =  100000000;
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int M, N;
    cin >> M >> N;

    // fertileなカラムを1としてビット化
    vi bitf(M, 0);
    rep(r, M) {
        int t = 0;
        rep(c, N) {
            int a;
            cin >> a;
            t |= (a << c);
        }
        bitf[r] = t;
    }

    // dp[i][j] := i番目の行をjにした時のパターン数
    // 前の行しか必要ないので2にする
    vector<vi> dp(2, vi(1<<N, 0));
    int cur = 0, next = 1;
    dp[0][0] = 1;

    rep(i, M) {
        fill(all(dp[next]), 0);
        rep(from, 1 << N) {
            if(dp[cur][from] == 0) continue;
            rep(to, 1 << N) {
                // infertileの箇所が1だったらだめ
                if ((to & bitf[i]) != to) continue;

                // 隣接して立ってるビットがあったらだめ
                if( (to & (to<<1)) != 0) continue;

                // 1つ上の行で隣接要素があったらだめ
                if ((from & to) != 0) continue;

                dp[next][to] = ((ll)dp[next][to] + dp[cur][from]) % MOD;
            }
        }
        swap(cur, next);
    }

    ll res = 0;
    rep(i, 1<<N)
        res = (res + dp[cur][i]) % MOD;
    cout << res << endl;

    return 0;
}
```

## POJ2836 - Rectangular Covering
2点以上をカバーする長方形にしないといけないので、点のままでは上手く処理できない。
まず2点を結んで長方形を作り、それが実際にはどの点をカバーするかを記録する。
あとはdp[カバーされた点]を用意し、各長方形を使ってビットDPすればよい。
なお、使用する長方形が増えると、カバーされた点の整数表記は必ず増加するので、DAGになりDPできることがわかる。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    while(cin >> n, n) {
        vector<pii> points(n);
        rep(i,n) cin >> points[i].first >> points[i].second;

        vector<pii> rects;   // (カバーしている点のビット表記, 領域サイズ)
        rep(i, n) {
            REP(j, i+1, n) {
                int x1 = min(points[i].first, points[j].first);
                int x2 = max(points[i].first, points[j].first);
                int y1 = min(points[i].second, points[j].second);
                int y2 = max(points[i].second, points[j].second);

                int w = x2 - x1;
                int h = y2 - y1;

                int covered = ( (1<<i) | (1<<j) );
                int area = w * h;
                // 横並びの場合
                if(area == 0) area = max(w, h);

                // 途中に含んでいる点を探す
                rep(k, n) {
                    if(k != i && k != j) {
                        if( x1 <= points[k].first && points[k].first <= x2 &&
                                y1 <= points[k].second && points[k].second <= y2)
                            covered |= (1<<k);
                    }
                }

                rects.push_back(mp(covered, area));
            }
        }

        // bit DP
        vi dp(1<<n, INF);
        dp[0] = 0;
        rep(S, 1<<n) {
            rep(i, rects.size()) {
                dp[S | rects[i].first] =
                        min(dp[S | rects[i].first], dp[S] + rects[i].second);
            }
        }

        cout << dp[(1<<n) - 1] << endl;
    }

    return 0;
}
```


## POJ1795 - DNA Laboratory
まず、文字列同士をくっつける順番を求める問題だと捉える。
例外として、ある文字列に完全に含まれている場合はくっつける必要が無いため除外しておく。
今まで使った文字列の集合と、最後に使った文字列が分かれば漸化式が立てられる。
このDPを解いた後、経路復元をすれば良い。

……が、辞書順最小の答えを求めるのが難しい。
本題とは外れるため、DPを解く所まで実装しそこで中断。
途中まで書いたものは[ここで正解っぽいことを確認した](http://quiz.fuqinho.net/blog/2012/06/15/poj-1795-dna-laboratory/)。


## POJ3411 - Paid Roads
ほぼ巡回セールスマン問題だが、親切なサンプルが伝えている通り、「行って帰ってくる」場合が最適なことがある。
そのため、DAGにならず普通にDPすると解けない。
というわけで、BFSして対応した。
一発で通せて楽しい。

```C++

struct Edge {
    int from, to, c, p, r;
};

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, m;
    cin >> N >> m;

    vector<Edge> edges(m);
    rep(i, m) {
        Edge e;
        cin >> e.from >> e.to >> e.c >> e.p >> e.r;
        --e.from, --e.to, --e.c;
        edges[i] = e;
    }

    // dp[S][v] := 今までに通った頂点の集合(S), 現在の頂点v による最小コスト
    vector<vi> dp(1<<N, vi(N, INF));
    dp[1<<0][0] = 0;

    queue< pii > que;
    que.push(mp(1<<0, 0));

    // bfs
    while(!que.empty()) {
        pii q = que.front(); que.pop();
        int S = q.first;
        int v = q.second;
        if (dp[S][v] == INF) continue;

        tr(it, edges) {
            if (v != it->from) continue;

            int next_bit = (1 << it->to);
            int c_bit = (1 << it->c);

            int addition = it->r;
            if ((S & c_bit) != 0) {  // もしRを使える条件を満たしているなら
                addition = min(addition, it->p);
            }

            int prev_val = dp[S | next_bit][it->to];
            dp[S | next_bit][it->to] =
                    min(dp[S | next_bit][it->to], dp[S][v] + addition);

            if(prev_val != dp[S|next_bit][it->to])
                que.push(mp(S|next_bit, it->to));
        }
    }

    int res = INF;
    rep(S, 1<<N)
        res = min(res, dp[S][N-1]);

    if(res == INF)
        cout << "impossible" << endl;
    else
        cout << res << endl;

    return 0;
}
```


## POJ3420 - Quad Tiling
[ここを見た](http://d.hatena.ne.jp/komiyam/20120812/1344733211)
[ここを見た](http://www.hankcs.com/program/algorithm/poj-3420-quad-tiling.html)

参考ページに頼るも理解できず。
計算途中に使われる変数のそれぞれが何を表しているか、vec[(1<<n)-1]が何故答えになるのかが不明……。


## POJ3735 - Training little cats
定数項を追加して、各処理をまとめたものを1つの行列にする。
グラフィックスの変換行列っぽいので、ゲームプログラムを書いたことがあれば分かりやすいかも。

g,sの処理は分かりやすいが、eの処理はその行全てを0にしないといけないことに注意。
「現在持っている個数」が定数項だけで表されるものではないからである。

なお、上までは順調に書けたが、行列同士の掛け算でTLEしてしまった。
[ここを参考に](http://area.hateblo.jp/entry/2013/03/18/013421)、掛け算の部分で一工夫を加えるとACになった。

```C++
class Matrix {
public:
    vector<vector<ll> > data;
    int n;

    Matrix(int n) : data(n, vector<ll>(n, 0)) {
        // 最初は単位行列にしておく.
        rep(i, n)
            data[i][i] = 1;
        this->n = n;
    }

    void zero() {
        rep(i, n)
            fill(all(data[i]), 0);
    }

    // mat * mat
    void times(Matrix &mat) {
        Matrix tmp(n);
        tmp.zero();

        rep(i, n) {
            rep(k, n) {
                if(data[i][k] != 0) {  // 0が結構あるので
                    rep(j, n) {
                        tmp.data[i][j] += data[i][k] * mat.data[k][j];
                    }
                }
            }
        }

        // copy
        rep(i, n) rep(j, n) data[i][j] = tmp.data[i][j];
    }

    // mat * vec
    vector<ll> times(vector<ll> &vec) {
        vector<ll> res(n, 0);
        rep(i, n) {
            rep(j, n) {
                res[i] += data[i][j] * vec[j];
            }
        }
        return res;
    }

    void dbg() {
        cout << "------" << endl;
        rep(i, n) {
            rep(j, n) {
                cout << data[i][j] << " ";
            }
            cout << endl;
        }
        cout << "------" << endl;
    }
};

// fpow for mat
Matrix fpow(Matrix &mat, ll k) {
    Matrix res(mat.n);
    Matrix x = mat;
    while (k != 0) {
        if ((k & 1) == 1)
            res.times(x);
        x.times(x);
        k >>= 1;
    }
    return res;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n, m, k;
    while (scanf("%d%d%d", &n, &m, &k), (n | m | k)) {
        Matrix whole(n + 1);
        rep(i, k) {
            char _command[3];
            scanf("%s", _command);
            char command = _command[0];

            if (command == 'g' || command == 'e') {
                int cat;
                scanf("%d", &cat);
                if(command == 'g')
                    whole.data[cat - 1][n]++;
                else
                    fill(all(whole.data[cat - 1]), 0);
            }
            else {
                int s1, s2;
                scanf("%d%d", &s1, &s2);
                swap(whole.data[s1 - 1], whole.data[s2 - 1]);
            }
        }

        // 行列累乗.
        Matrix result = fpow(whole, m);

        rep(i, n) {
            if (i != 0)
                printf(" ");
            printf("%lld", result.data[i][n]);
        }
        printf("\n");
    }

    return 0;
}
```


## POJ3171 - Cleaning Shifts
[ここを見た](http://d.hatena.ne.jp/komiyam/20120813/1344786827)
区間中の最小値を探すのにセグメント木を使って高速化する方法。
後ろから更新してゆくことで上手く処理できる。
セグメント木をLong Longに対応させるので手こずった。
また、mapを使っても大丈夫かと思ったがTLEした。
かなり速度に差がありそう。

```C++
const ll INF = (ll)1 << 60;
class SegTree {
    int n;
    vector<ll> dat;

public:
    SegTree(int _n) {
        n = 1;
        while (n < _n) n *= 2;
        dat.resize(2 * n - 1);
        fill(all(dat), INF);
    }

    void update(int k, ll a) {
        k += n - 1;
        dat[k] = a;
        while (k > 0) {
            k = (k - 1) / 2;
            dat[k] = min(dat[k * 2 + 1], dat[k * 2 + 2]);
        }
    }

    ll _query(int a, int b, int k, int l, int r) {
        if (r <= a || b <= l) return INF;

        if (a <= l && r <= b) return dat[k];
        else {
            ll vl = _query(a, b, k * 2 + 1, l, (l + r) / 2);
            ll vr = _query(a, b, k * 2 + 2, (l + r) / 2, r);
            return min(vl, vr);
        }
    }

    // [a, b)
    ll query(int a, int b) {
        return _query(a, b, 0, 0, n);
    }

    // a
    ll query(int a) {
        return query(a, a+1);
    }
};

const int MAX_T = 86399 + 10;
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, M, E;
    scanf("%d%d%d", &N, &M, &E);
    ++E;

    // mapを使うとTLEした.
    vector<pii> cows[MAX_T];
    int max_t = 0;
    rep(i, N) {
        int t1, t2, s;
        scanf("%d%d%d", &t1, &t2, &s);
        ++t2;
        cows[t1].push_back(mp(t2, s));

        max_t = max(max_t, t2);
    }

    SegTree seg(max_t + 2);  // i以降を全て被覆するのに必要な最小コスト.
    seg.update(E, 0);
    for (int i = E; i >= M; --i) {  // 後ろから更新.
        tr(it, cows[i]) {
            ll mini = seg.query(i, it->first + 1);  // [i, end+1]
            seg.update(i, min(seg.query(i), mini + it->second));
        }
    }

    ll res = seg.query(M);
    printf("%lld\n", (res==INF) ? -1 : res);

    return 0;
}
```


## まとめ
ビットDPは比較的とっつきやすかった。
基本的に力技の全探索なので、イメージは掴みやすい。
行列累乗も、DirectXやOpenGLの変換行列と考えは同じなので、漸化式さえ分かれば倒せそう。

練習問題一覧（ネットワークフロー前まで）をひとしきり回れたので、そろそろ実践の問題を解いてゆきたい。
練習問題も併せて進める予定。
ここまで長かった……。
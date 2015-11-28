title: "Ari-practice solve 12"
date: 2015-11-11 00:06:20
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (12)
BIT、セグメント木とバケット法（川柳）。
よく名前は聞いていたけど、ちゃんと解いたことは無かった。

## POJ1990 - MooFest
[ここを見た](http://torus711.hatenablog.com/entry/20121114/p1)
BITが蟻本を読んでもイマイチピンと来なかったが、この問題でイメージが掴めた気がする。
最初の練習問題にしては難しい気も……。

まず、BIT関係なく牛を処理する方法について。
聴力の昇順にソートして順に処理してゆくと、それまでに処理した牛は今処理している牛より聴力値が低い。
そのため、今の牛との座標の差の和さえ分かれば答えとして加算する値が求まる。

| 聴力 | 座標 |
|:-----------|:------------|
| 2 | 5 |
| 2 | 6 |
| 3 | 1 |
| 4 | 3 |

という順に並んでいれば、以下の式で答えが求まる。

$$0 + 2*1 + 3*(4+5) + 4 * (2+3+2)$$

この座標の差を効率的に求めるため、2つのBITを用意する。
「座標xまでにいる牛の数の和」「座標xまでの座標値の和」をそれぞれBITで管理すると、座標を与えればその座標までの「牛の数」「座標値の和」が分かるようになる。
左右それぞれの情報が取り、これを利用すれば座標の差が求まる。
詳細は参考URLで。

これにより、$O(nlogn)$で答えが計算できるようになる。

BITのコードは省略。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    vector<pii> vol_pos(N);
    int max_pos = 0;
    rep(i, N) {
        cin >> vol_pos[i].first >> vol_pos[i].second;
        max_pos = max(max_pos, vol_pos[i].second);
    }
    sort(all(vol_pos));

    BIT bit_pos(max_pos), bit_pos_sum(max_pos);

    ll res = 0;
    rep(i, N) {
        ll left_cow_num = bit_pos.sum(vol_pos[i].second);
        ll right_cow_num = bit_pos.sum(max_pos) - left_cow_num;

        ll left_cow_pos_sum = bit_pos_sum.sum(vol_pos[i].second);
        ll right_cow_pos_sum = bit_pos_sum.sum(max_pos) - left_cow_pos_sum;

        ll left_sum = vol_pos[i].first *
                      (vol_pos[i].second * left_cow_num - left_cow_pos_sum);
        ll right_sum = vol_pos[i].first *
                      (right_cow_pos_sum - vol_pos[i].second * right_cow_num);
        res += left_sum + right_sum;

        bit_pos.add(vol_pos[i].second, 1);
        bit_pos_sum.add(vol_pos[i].second, vol_pos[i].second);
    }

    cout << res << endl;

    return 0;
}
```


## POJ3109 - Inner Vertices
[ここを見た](http://even-eko.hatenablog.com/entry/2013/10/07/233649)
まだまだBITに慣れていないためか、自分で思いつく気がしなかった。

### 圧縮
座標圧縮をする。
蟻本ではfindで置き換える値を探しているが、にぶたんできるので書き換えて高速化。
BITでは1-basedのindexが扱いやすいので、1を足した。

### 考え方
y座標の小さい順に（下から）見てゆく。
点に挟まれている範囲をBITに加算。
これにより、縦に挟まれた際に、その間に横にも挟まれている点の個数が高速に見つけられる。

範囲加算に対するBITの利用は蟻本通り。
なお、もう一度出てきたら答えに加算するため、出現したx座標をメモしている。

ちなみに、cin/coutを使うとTLEした。

```C++
// BIT
struct BIT {
    int n;
    vi bit;

    BIT(int n): n(n), bit(n+1, 0) {
    }

    int sum(int i) {
        int s = 0;
        while (i > 0) {
            s += bit[i];
            i -= i & -i;
        }
        return s;
    }

    void add(int i, int x) {
        while (i <= n) {
            bit[i] += x;
            i += i & -i;
        }
    }

    void range_add(BIT& bit1, int l, int r, int x) {
        add(l, -x * (l-1));
        bit1.add(l, x);
        add(r, x*r);
        bit1.add(r, -x);
    }
};


int compress(vi& x1) {
    vi xs;
    rep(i, x1.size()) {
        xs.push_back(x1[i]);
    }
    sort(all(xs));
    xs.erase(unique(all(xs)), xs.end());

    rep(i, x1.size())
        x1[i] = distance(xs.begin(), lower_bound(all(xs), x1[i])) + 1;

    return xs.size();
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    scanf("%d", &n);

    vi x(n), y(n);
    rep(i, n)
        scanf("%d %d", &x[i], &y[i]);
    int w = compress(x);
    int h = compress(y);

    // yを基準とした配列に
    vector<vi> points(h+1);
    rep(i, n)  points[y[i]].push_back(x[i]);

    vector<bool> covered(w+1, false);
    BIT bit0(w), bit1(w);

    int ans = n;
    rep(y, h+1) {
        sort(all(points[y]));
        int xnum = points[y].size();

        rep(i, xnum) {
            int x = points[y][i];

            // xの値を取る
            int x_val = bit0.sum(x) + bit1.sum(x) * x;
            x_val -= bit0.sum(x - 1) + bit1.sum(x - 1) * (x - 1);

            // 縦に見て既に1つ以上あった場合
            if(covered[x])
                ans += x_val;
            covered[x] = true;

            // 足した分は除外
            bit0.add(x, -x_val);

            if(i < xnum - 1) {  // 右端でない
                // i <-> (i+1) 間にある個数を足す
                int x2 = points[y][i + 1];
                bit0.range_add(bit1, x+1, x2-1, 1);
            }

        }
    }

    printf("%d\n", ans);

    return 0;
}
```


## POJ2155 - Matrix
前提として、「notを取って0/1にする」のは「加算して%2を取る」のと同じ。
単に足し算をすればいいことが分かる。

解き方として、二次元BITだろうと当たりは付くが、物は試し、一次元BITで通るか試してみた。

### 一次元BIT
y座標のそれぞれにBITを用意して、x座標の範囲和を素早く求める。
計算量は、$O(XTNlogN)$。
どう見ても間に合わないが、やっぱりTLEした。
一応コードを掲載。

```C++
// TLE!
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int __test;
    scanf("%d", &__test);
    while(__test--) {
        int N, T;
        scanf("%d%d", &N, &T);

        vector<BIT> xbit0(N+1, BIT(N)), xbit1(N+1, BIT(N));

        rep(_query, T) {
            char c[2];
            scanf("%s", c);

            if(c[0] == 'C') {
                int x1, y1, x2, y2;
                scanf("%d%d%d%d", &x1, &y1, &x2, &y2);

                REP(y, y1, y2+1) {
                    xbit0[y].range_add(xbit1[y], x1, x2, 1);
                }
            }
            else {
                int x, y;
                scanf("%d%d", &x, &y);
                printf("%d\n",
                       xbit0[y].range_sum(xbit1[y], x, x) % 2 );
            }
        }
    }

    return 0;
}
```

### 二次元BIT
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/09/08/231104)

蟻本にはコードの例が無かったので自分で書いてみたが、イマイチ答えが合わない。
Webを漁ってみると、BITを4つ使った答えが多く、これが本来の実装なのかもしれない。
ただ、この問題であれば、上記ブログにあるようにBITは1つでいいはず。

「(xl,yl)->(xr,yr)」の領域を求めたい。
「(0,0)->(xl,yl)」をまず足し、「(0,0)->(xr,yr)」を足す。
すると、「(0,0)->(xl,yl)」は2回足され、それ以外の領域が1回足されている。
そのため、「(0,0)->(xr,yl)」「(0,0)->(xl,yr)」（境界は雑）を引けば、求めたい領域のみの総和を得ることが出来る。

加算は、上記を実現するように4箇所足し込む。
左上/右下は+、左下/右上は-にすると良い。
総和は普通のBITと同様（2次元にはしている）に求めれば良い。

```C++
// BIT x BIT
struct BIT2 {
    int n;
    vector<vi> bit;

    BIT2(int n) : n(n), bit(n + 1, vi(n + 1)) {
    }

    int sum(int y, int x) {
        int s = 0;
        while (y > 0) {
            int tx = x;
            while (tx > 0) {
                s += bit[y][tx];
                tx -= tx & -tx;
            }
            y -= y & -y;
        }
        return s;
    }

    void add(int y, int x, int val) {
        while (y <= n) {
            int tx = x;
            while (tx <= n) {
                bit[y][tx] += val;
                tx += tx & -tx;
            }
            y += y & -y;
        }
    }
};


int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int __test;
    scanf("%d", &__test);
    while(__test--) {
        int N, T;
        scanf("%d%d", &N, &T);

        BIT2 bit(N);

        rep(_query, T) {
            char c[2];
            scanf("%s", c);

            if (c[0] == 'C') {
                int x1, y1, x2, y2;
                scanf("%d%d%d%d", &x1, &y1, &x2, &y2);

                // 調整
                x2++, y2++;

                bit.add(y1, x1, 1);
                bit.add(y1, x2, -1);
                bit.add(y2, x1, -1);
                bit.add(y2, x2, 1);
            }
            else {
                int x, y;
                scanf("%d%d", &x, &y);
                cout <<  bit.sum(y, x) % 2 << endl;
            }
        }

        if(__test > 0)
            cout << endl;
    }

    cerr << -1 % 2 << endl;
    return 0;
}
```


## POJ2886 - Who Gets the Most Candies?
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/09/09/215038)

なんとも厄介な問題。

まず、約数は普通に求めると間に合わないので、篩のように処理をすると$O(N)$で求まるっぽい。
詳しくはコード参照。

次に、「子供が抜けていく」という状況は、BITを使うと高速に処理できる。
イメージとしては、「1 2 3 4 5」という5人の子供が居る場合に3番目が抜けた時、「1 2 2 3 4」となるようにBITの3番目に-1を足す。
こうすると、2番目は左側の2だと分かるため、二分探索でここを探せば早い。
間の要素が抜けた後の処理が高速に行えるため、色々な問題で使えそうだ。

何故か、子供の名前をstringで受け取るとTLEした。
わけがわからないよ……。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);


    // 約数を求める
    const int MAX_N = int(1e5) * 5 + 10;
    vi divNum(MAX_N, 1);
    REP(i, 2, MAX_N) {
        if(divNum[i] == 1) {  // 素数なら
            for(int j = i; j <= MAX_N; j += i) {
                int k = 0;
                for(int m=j; m%i==0; m/=i, k++);
                divNum[j] *= k+1;
            }
        }
    }

    int N, K;
    while(cin >> N >> K) {
        K--;

        BIT bit(N);

        vector< pair<char[11], int> > children(N);
        rep(i, N)
            cin >> children[i].first >> children[i].second;

        // 0 -> (N-1) のBITを用意
        rep(i, N)
            bit.add(i, 1);

        pair<string, int> ans = mp("", -1);
        int now = K, next = now;
        REP(out, 1, N + 1) {
            if (ans.second < divNum[out])
                ans = mp(children[now].first, divNum[out]);

            // 1 2 3 4 -> 1 2 2 3 のようになる
            bit.add(now, -1);

            if (out == N)
                break;

            int mod = (N - out);
            next += children[now].second;
            if (children[now].second > 0)
                next--;
            next %= mod;

            // マイナスなら調整
            if (next < 0)
                next += mod;

            // 左端を探す
            int lb = -1;  // 答えの左隣
            int ub = N;   // 答え
            while (ub - lb > 1) {
                int med = (lb + ub) / 2;
                if (bit.sum(med) < next + 1)  // nextは0-baseなので、lb="1つ前の値の右端"
                    lb = med;
                else
                    ub = med;
            }

            now = ub;
        }

        cout << ans.first << " " << ans.second << endl;
    }
}
```

## POJ3264 - Balanced Lineup
普通のSegTreeを、最小値と最大値を保持するよう変形すればOK。
ちなみに、cin/coutではTLEした。

```C++
class SegTree {
    int n;
    vector<pii> dat;

public:
    SegTree(int _n) {
        n = 1;
        while(n < _n) n *= 2;
        dat.resize(2*n-1);
        fill(all(dat), mp(INT_MAX, 0));
    }

    pii cond(pii t1, pii t2) {
        return mp( min(t1.first, t2.first), max(t1.second, t2.second) );
    }

    void update(int k, pii a) {
        k += n - 1;
        dat[k] = a;
        while(k > 0) {
            k = (k-1) / 2;
            dat[k] = cond(dat[k*2+1], dat[k*2+2]);
        }
    }

    pii _query(int a, int b, int k, int l, int r) {
        if(r <= a || b <= l) return mp(INT_MAX, 0);

        if(a <= l && r <= b) return dat[k];
        else {
            pii vl = _query(a, b, k*2+1, l, (l+r)/2);
            pii vr = _query(a, b, k*2+2, (l+r)/2, r);
            return cond(vl, vr);
        }
    }
    // [a, b)
    pii query(int a, int b) {
        return _query(a, b, 0, 0, n);
    }
};

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, Q;
    scanf("%d%d", &N, &Q);

    SegTree seg(N);
    rep(i, N) {
        int c;
        scanf("%d", &c);
        seg.update(i, mp(c,c));
    }

    rep(i, Q) {
        int a, b;
        scanf("%d%d", &a, &b);
        --a;
        pii res = seg.query(a, b);
        printf("%d\n", res.second - res.first);
    }

    return 0;
}
```


## POJ3368 - Frequent values
[ここを見た](http://torus711.hatenablog.com/entry/20121117/p1)
区間内で頻出の数値を探す。
区間はソートされているので、「頻出の数値」を区間に置き換えることができる。
セグメント木を使って、各要素に「その区間内での最頻出数値の出現回数」「その区間内での左端の値の出現回数」「右〃」の3つを持たせ、上手く初期化＆マージしてゆく。

蟻本と同様に書いていたら、nを2のべき乗にしていたため範囲外アクセスをしてしまっていた。
要確認。

```C++
class SegTree {
    int n;
    vi orig;
    vi inside;  // 区間[k]内での最長
    vi left;    // 区間[k]の左端の長さ
    vi right;

    // [1 1 2 2 2 3 4] なら、
    // inside[k] => 3
    // left[k] => 2
    // right[k] => 1

public:
    SegTree(int _n, vi& _a) {
        orig = _a;
        n = _n;

        inside.resize(4*n);
        left.resize(4*n);
        right.resize(4*n);
        fill(all(inside), 1);
        fill(all(left), 1);
        fill(all(right), 1);
    }

    // [l, r)
    void init(int k, int l, int r) {
        if(r - l == 1) {  // 幅1 葉っぱ
            inside[k] = left[k] = right[k] = 1;
            return;
        }

        int child1 = k * 2 + 1, child2 = k * 2 + 2, mid = (l + r) / 2;
        init(child1, l, mid);
        init(child2, mid, r);

        inside[k] = max(inside[child1], inside[child2]);
        left[k] = left[child1];
        right[k] = right[child2];

        // 左の子の右端 == 右の子の左端  => 真ん中はくっつけられる
        if(orig[mid - 1] == orig[mid])
            inside[k] = max(inside[k], right[child1] + left[child2]);

        // 両方の子の左端が同じ => 左側の子は全て同じ要素
        if(orig[l] == orig[mid])
            left[k] += left[child2];
        // ↑と同じ
        if(orig[mid - 1] == orig[r])
            right[k] += right[child1];
    }

    struct Res { int i, l, r; };
    Res _query(int a, int b, int k, int l, int r) {
        if(r <= a || b <= l) return Res{0, 0, 0};

        if(a <= l && r <= b) {
            return Res{inside[k], left[k], right[k]};
        }
        else {
            int child1 = k*2+1, child2 = k*2+2, mid = (l+r)/2;
            Res vl = _query(a, b, child1, l, mid);
            Res vr = _query(a, b, child2, mid, r);

            // initと同じことをする
            Res res{
                    max(vl.i, vr.i),
                    vl.l,
                    vr.r
            };
            if(orig[mid-1] == orig[mid])
                res.i = max(res.i, vl.r + vr.l);
            if(orig[l] == orig[mid])
                res.l += vr.l;
            if(orig[mid-1] == orig[r])
                res.r += vl.r;

            return res;
        }
    }
    // [a, b)
    int query(int a, int b) {
        return _query(a, b, 0, 0, n).i;
    }
};

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, Q;
    while( scanf("%d", &N), N ) {
        scanf("%d", &Q);
        vi a(N);
        rep(i,N) {
            int t;
            scanf("%d", &t);
            a[i] = t;
        }

        SegTree seg(N, a);
        seg.init(0, 0, N);

        while(Q--) {
            int l, r;
            scanf("%d%d", &l, &r);
            --l;
            printf("%d\n", seg.query(l, r));
        }
    }

    return 0;
}
```


## POJ3470 - Walls
解き方が分からず、日本語の解説も無いので断念。

## POJ1201 - Intervals
[ここを見た](http://quiz.fuqinho.net/blog/2012/06/12/poj-1201-intervals/)

まさかのグラフ解法で、テーマには合っていないが……。
確かによく見ると、蟻本にも掲載されているPOJ3169と同じ問題。
セグメント木で解けるのかもしれないが、記事がみつからないので断念。

いやーしかし、このグラフの解き方は毎回感心してしまう。
不等式をグラフに当てはめ、最短経路問題にするというのは垣根を超えた感がすごい。
競プロのプロはきっとこんな事を平然とやってのけるのだろう。

```C++
struct Edge { int from,to,cost; };

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    cin >> n;

    vector<Edge> es;
    int mxv = 0;
    rep(i, n) {
        int a, b, c;
        cin >> a >> b >> c;
        mxv = max(mxv, max(a, b));
        // b -> a (cost: -c)
        es.push_back(Edge{b, a-1, -c});
    }
    mxv++;

    // i -> (i+1) (cost: 1)
    // (i+1) -> i (cost: 0)
    rep(i, mxv) {
        es.push_back(Edge{i, i+1, 1});
        es.push_back(Edge{i+1, i, 0});
    }

    // bellman-ford
    vi d(mxv+1, 0);
    while(true) {
        bool update = false;
        tr(it, es) {
            if(d[it->from] != INF && d[it->to] > d[it->from] + it->cost) {
                d[it->to] = d[it->from] + it->cost;
                update = true;
            }
        }
        if(!update) break;
    }

    cout << -1 * d[0] << endl;
    return 0;
}
```

## UVa11990 - Dynamic Inversion
[ここを見た](http://quiz.fuqinho.net/blog/2012/06/14/uva-11990-dynamic-inversion/)

参考URLを丸写しした。
平方分割はスラスラ書けるには慣れが必要な気がする。
考え方としてはそこまで難しい訳ではないが、実装が意外と重たくかなり時間が掛かった。


## まとめ
かなり重たい回だった。

BIT/セグメント木は考え方がイメージしにくく、解き方が全く浮かばない状態から参考URLを写経する作業になりがちだった。
ただ、「こんなことも出来るのか」という発見の連続だったので楽しいといえば楽しかった。

数列を扱う問題に適用されるアルゴリズムたちだが、そのような問題は多い（コドフォDiv.2のD,Eあたり？）ので活用範囲は広そう。
特に難しい問題の高速化に使われる事が多いと思われる。

ちなみに、今の状態だと未だ参考URLを見ずには実装できないので、問題の認識力を上げた段階に留まっている。
実装練習が後々必要だ……。
title: "Codeforces Wunder Fund Round 2016 A-D"
date: 2016-02-11 22:37:00
tags:
- esplo77
- Codeforces
- C++11
---

## A. Slime Combining
[問題文](http://codeforces.com/contest/618/problem/A)

### 考え方
1を右端に置いてゆき、同じ数字が連続したらその値+1される。
愚直に実装すればよいが、TLEが怖かったのでコンテスト中は2進数として捉えて実装した。

```C++
vi ans;
void rec(int n) {
    if(n==0)
        return;

    int init = 1;
    int p = 0;
    rep(i, 30) {
        if(n < init) {
            p = i-1;
            break;
        }
        else if(n == init) {
            p = i;
            break;
        }
        init *= 2;
    }
    ans.push_back(p+1);
    int rest = n - (int)pow(2, p);
    return rec(rest);
}

int main() {
    int n;
    cin >> n;

    rec(n);

    tr(it, ans)
        cout << *it << " ";
    cout << endl;
	
    return 0;
}
```


## B. Guess the Permutation
[問題文](http://codeforces.com/contest/618/problem/B)

### 考え方
小さい方の値を取っていって作られた行列なので、比較対象に自分より大きい要素があれば、必ず自分が出現する。
そのため、1番目の要素を知りたいのであれば、行列の1行目から最大値を取り出せば良い。
ただ、元の配列での最大値だけは行列に出現しないため、二箇所で算出される最大値-1のいずれかを最大値とみなせばよい。

```C++
int main() {
    int n;
    cin >> n;

    vi ans;
    bool firstN = true;
    rep(i, n) {
        int mx = 0;
        rep(j, n) {
            int t;
            cin >> t;
            chmax(mx, t);
        }
        if(mx == n-1 && firstN) {
            mx++;
            firstN = false;
        }
        ans.push_back(mx);
    }

    tr(it, ans)
        cout << *it << " ";
    cout << endl;
	
    return 0;
}
```


## C. Constellation
[問題文](http://codeforces.com/contest/618/problem/C)

### 考え方
他の点を内部に含まない三角形を作る問題。
任意の点を取り、そこから近い順に2点を取れば、他の点は中に無い。
そのため、1つ適当な点を取り、その点との距離でソートし、近い順に取れば良い。

ただし、コーナーケースとして、3点が直線上に並ぶ場合がある。
この場合面積が0になるので、その場合は次に近い点を取る。
まず最短の2点を結ぶと、その直線上に並ぶ点は、直線上に無い点との三角形には含まれない。
そのため、近い順に試せば問題がないことがわかる。

なお、三角形の面積は成分による内積で判定すると楽。

```C++
ll calc_dist(pii p1, pii p2) {
    ll x = p1.first - p2.first;
    ll y = p1.second - p2.second;
    return x*x + y*y;
}

int main() {
	int n;
    cin >> n;

    vector<pii> points(n);
    rep(i, n) cin >> points[i].first >> points[i].second;

    vector<pair<ll,int> > dist(n-1);
    REP(i, 1, n)
        dist[i-1] = mp(calc_dist(points[0], points[i]), i);
    sort(all(dist));

    int two = dist[0].second;

    pair<ll,ll> A = mp(points[two].first - points[0].first,
                       points[two].second - points[0].second);

    REP(_i, 1, n-1) {
        int ind = dist[_i].second;

        pair<ll,ll> B = mp(points[ind].first - points[0].first,
                           points[ind].second - points[0].second);

        if( A.first * B.second - A.second * B.first != 0) {
            cout << 1 << " " << two+1 << " " << ind+1 << endl;
            return 0;
        }
    }
	
    return 0;
}
```


## D. Hamiltonian Spanning Tree
[問題文](http://codeforces.com/contest/618/problem/D)

### 考え方
問題は2パターンに分かれる。

#### x >= y
この場合、なるべくSpanning Treeを使わない方が良い。
基本的に全てyが使えるが、あるノードが全ノードへのエッジを持っている場合、1つはxを使わないといけないことに注意。

#### x < y
この場合はなるべくSpanning Treeを使う。
動きとしては、全域木を葉から葉へたどり、通常のエッジ（y）を通り、また全域木に戻り同じことをする……となる。

そのため、子要素が2以上あるノードは+2、子要素が1つのノードは+1、葉は+0として計算すると、通ることのできるエッジ数が分かる。
[公式の解説](http://codeforces.com/blog/entry/23142)

```C++
vvi graph;
int ans;

int dfs(int v, int pre) {
    int left = 2;  // 子要素の数
    tr(it, graph[v]) {
        if (*it == pre)
            continue;
        int x = dfs(*it, v);
        if(left > 0 && x == 1) {
            ans++;
            left--;
        }
    }

    return min(left, 1);
}

int main() {
    int n;
    ll x, y;
    cin >> n >> x >> y;

    graph.resize(n);
    rep(i, n - 1) {
        int a, b;
        cin >> a >> b;
        --a, --b;
        graph[a].push_back(b);
        graph[b].push_back(a);
    }

    // なるべくyをつかう
    if (x >= y) {
        int mx = 0;
        rep(i, n)
            chmax(mx, (int) graph[i].size());

        if (mx == n - 1)  // 必ず1つxを使わないといけない
            cout << x + y * (n - 2) << endl;
        else
            cout << y * (n - 1) << endl;
        return 0;
    }

    // なるべくxをつかう
    dfs(0, -1);
    cerr << ans << endl;
    cout << x * ans + y * (n - 1 - ans) << endl;

    return 0;
}
```


## まとめ
参加者は3757人。

- A: 3552/3667
- B: 3041/3159
- C: 972/2469
- D: 281/1151
- E: 85/139
- F: 11/41
- G: 0/1

Cを落としている人が多く、Hackチャンスだったが活かせなかった。
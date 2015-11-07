title: "Ari-practice solve 11"
date: 2015-11-08 03:06:20
tags:
- esplo77
- C++
- 蟻本
---


# 蟻本練習問題を解いた (11)
色々な実装。

## POJ2566 - Bound Found
[ここを見た](http://d.hatena.ne.jp/komiyam/20120806/1344179663)
単純なしゃくとりではなく、工夫が必要。
このままではしゃくとりが出来ないので、累積和を取りソートする。
これにより、「元の範囲を一意に特定でき」「しゃくとりが適用できる条件を満たした」配列が出来上がる。
ソートによって（元の順番で言う）左右が入れ替わっている場合もあり、これを許容することで絶対値の条件をこなしているのが中々思いつかなかった。

求めたいのはsums[u]-sums[l]なので、l,uを移動させるたびに答えを更新している。
更新処理を関数に括りだすと、見た目が普通のしゃくとりっぽくなって良い。

```C++
int t;

const int INF = 1 << 30;
int update(vector<pii>& sums,
           int l, int u,
           int& ans, pii& ans_lu) {
    int val = sums[u].first - sums[l].first;
    if(u <= l)  // 追いついた
        return -INF;
    if(abs(t-val) < abs(t-ans)) {  // 答えを更新
        ans = val;
        int mi = min(sums[l].second, sums[u].second);
        int mx = max(sums[l].second, sums[u].second);
        ans_lu = mp(mi, mx);
    }
    return val;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n, k;
    while(cin >> n >> k, (n|k)) {
        vi seq(n);
        rep(i, n)
            cin >> seq[i];

        // accumulate sum
        vector<pii> sums(n+1);
        sums[0] = mp(0, 0);
        rep(i, n)
            sums[i+1] = mp(sums[i].first + seq[i], i+1);
        sort(all(sums));

        while(k--) {
            cin >> t;

            int sum = -INF;
            int l = 0, u = 0;
            int ans = -INF;
            pii ans_lu;
            while(true) {  // 累積和でしゃくとり
                while(u < n && sum < t)
                    sum = update(sums, l, ++u, ans, ans_lu);
                if(sum < t)
                    break;
                sum = update(sums, ++l, u, ans, ans_lu);
            }

            cout << ans << " " << ans_lu.first+1 << " " << ans_lu.second << endl;
        }
    }

    return 0;
}
```


## POJ2739 - Sum of Consecutive Prime Numbers
こちらは普通のしゃくとり。
特に罠もないので、解くだけ。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int MAX_N = 10100;

    vi primes;
    vector<bool> is_prime(MAX_N, true);
    is_prime[0] = is_prime[1] = false;
    REP(i, 2, MAX_N) {
        for(int j=i+i; j < MAX_N; j+=i)
            is_prime[j] = false;
    }
    rep(i, MAX_N) {
        if(is_prime[i])
            primes.push_back(i);
    }

    int N = primes.size();

    int num;
    while(cin >> num, num) {
        int res = 0;
        int sum = 0, s=0, e=0;
        while(true) {
            while(e < N && sum < num)
                sum += primes[e++];
            if(sum < num)
                break;
            if(sum == num)
                res++;
            sum -= primes[s++];
        }

        cout << res << endl;
    }

    return 0;
}
```

## POJ2100 - Graveyard Design
またもシンプルなしゃくとり。
範囲は適当に$10^7$くらい取るとよい。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    ll n;
    cin >> n;

    const int MAX_LOOP=(int)1e7 + 100;

    ll sum=0;
    int lb=0, ub=0;
    vector<pii> res;
    while(true) {
        while(ub < MAX_LOOP && sum < n) {
            ++ub;
            sum += (ll)ub*ub;
        }
        if(sum < n)
            break;
        if(sum == n)
            res.push_back(mp(lb+1, ub));  // lbは既に削っているので
        ++lb;  // 0を含んではいけない
        sum -= (ll)lb*lb;
    }

    cout << res.size() << endl;
    rep(i, res.size()) {
        cout << res[i].second - res[i].first + 1 << " ";
        REP(j, res[i].first, res[i].second+1) {
            if( j != res[i].first )
                cout << " ";
            cout << j;
        }
        cout << endl;
    }

    return 0;
}
```


## POJ3185 - The Water Bowls
[ここを見た]()
左から順に見ていって反転させるだけでええやん。
と思ったら、左端が決定できないことを失念していた。
参考にしたサイトで、配列の左側を拡張するスマートな解き方があったため拝借した。
この問題はひっくり返す数が3に固定されているので、蟻本の方法を使わなくても問題はない（そもそもn=20なので余裕）。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    const int N = 20;

    vector<bool> bowls(N);
    rep(i, N){
        int t;
        cin >> t;
        bowls[i] = t != 1;  // 0:true, 1:false
    }

    // 左端は決定できないため、最初に0か1を付与して両方試す
    vi res(2, 0);
    rep(k,2) {
        vi cpy_bowls(N+1);
        rep(i, N) cpy_bowls[i+1] = bowls[i];
        cpy_bowls[0] = k != 1;

        rep(i, N) {
            if (!cpy_bowls[i]) {
                res[k]++;
                // 1つ右を中心にひっくり返す
                cpy_bowls[i] = true;
                cpy_bowls[i + 1] = !cpy_bowls[i + 1];
                if (i + 2 < N+1)
                    cpy_bowls[i + 2] = !cpy_bowls[i + 2];
            }
        }
    }

    cout << min(res[0], res[1]) << endl;

    return 0;
}
```


## POJ1222 - Extended Lights Out
蟻本の問題より、サイズが決まっている分簡単。
`vector<bool>`にはflip()が用意されている事を補完で初めて知った。

```C++
const int WIDTH = 6;
const int HEIGHT = 5;
void flip_around(vector< vector<bool> >& now, int i, int j) {
    if(j!=0) now[i][j-1].flip();
    now[i][j].flip();
    if(j<WIDTH-1) now[i][j+1].flip();

    if(i!=0) now[i-1][j].flip();
    if(i<HEIGHT-1) now[i+1][j].flip();
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int testcase;
    cin >> testcase;
    rep(_test, testcase) {
        cout << "PUZZLE #" << _test + 1 << endl;

        vector< vector<bool> > mat(HEIGHT, vector<bool>(WIDTH));
        rep(i, HEIGHT) rep(j, WIDTH) { bool b; cin >> b; mat[i][j] = b; }

        rep(first_row, 1<<WIDTH) {
            vector< vector<bool> > now(HEIGHT, vector<bool>(WIDTH)),
                    ans(HEIGHT, vector<bool>(WIDTH, false));
            rep(i, HEIGHT) rep(j, WIDTH) now[i][j] = mat[i][j];

            // flip first row
            rep(j, WIDTH) {
                if( ((first_row>>j) & 1) == 1 ) {
                    ans[0][j] = true;
                    flip_around(now, 0, j);
                }
            }

            // second - fifth line
            REP(i, 1, HEIGHT) {
                rep(j, WIDTH) {
                    // if the above cell has to flip
                    if(now[i-1][j]) {
                        ans[i][j] = true;
                        flip_around(now, i, j);
                    }
                }
            }

            if( accumulate( all(now[HEIGHT-1]), 0 ) == 0 ) {
                rep(i, HEIGHT) {
                    rep(j, WIDTH) {
                        if(j != 0)
                            cout << " ";
                        cout << ans[i][j];
                    }
                    cout << endl;
                }
            }
        }
    }

    return 0;
}
```


## POJ2674 - Linear world
[ここを見た](http://blue-jam.hatenablog.com/entry/2014/09/04/010925)
蟻本の問題とおおよそ同じ。
ソートをする必要はなく、最後に落ちるものより左側にあるものを求めれば答えがわかる。

時間制限が厳しので、scanf/printfに書き換えた。
%13.2lfで出力したらWAになり、かなりの時間を無駄にした。理由が分からん。

```C++
struct Person {
    int dir;
    double pos;
    char name[260];
};
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(2);
    ios_base::sync_with_stdio(false);

    int N;
    while(scanf("%d", &N), N) {
        double L, V;
        scanf("%lf%lf", &L, &V);

        vector<Person> inhab(N);

        double max_time = -1;
        int res_i = -1;
        int neg_count = 0;
        for(int i = 0; i < N; ++i){
            char buf[4];
            scanf("%s%lf%s", buf, &inhab[i].pos, inhab[i].name);
            inhab[i].dir = ( tolower(buf[0]) == 'p') ? 1 : -1;

            if(inhab[i].dir == -1)
                neg_count++;

            double tim = (L - inhab[i].pos) / V;
            if(inhab[i].dir == -1)
                tim = inhab[i].pos / V;

            if(tim > max_time) {
                max_time = tim;
                res_i = i;
            }
        }

        // 最後に落ちるものより左側に落ちるのは neg_count 個
        int res = neg_count;
        if(inhab[res_i].dir == -1)
            res--;
        printf("%13.2f %s\n", (int)(max_time*100)/100.0, inhab[res].name);
    }

    return 0;
}
```


## POJ3977 - Subset
おおよそ蟻本と同じ。
ただし、結果はabsなので少しややこしい。
lower_boundを使った際は、見つかった場所の1つ前もチェックしないといけないことに注意。

```C++
void power_set( vector<ll>& group, vector< pair<ll, int> >& pset ) {
    REP(i, 1, 1<<group.size()) {
        ll sum = 0;
        int count = 0;
        rep(j, 31) {
            if( ((i>>j)&1) == 1 ) {
                sum += group[j];
                count++;
            }
        }
        pset.push_back(mp(sum, count));
    }

    sort(all(pset));
    pset.erase(unique(all(pset)), pset.end());
}
ll myabs(ll a) {
    return (a < 0) ? -a : a;
}

void find_near(
        vector<pair<ll, int> >& target,
        pair<ll, int> num,
        vector< pair<ll, int> >& res) {
    vector<pair<ll, int> >::iterator it = lower_bound(all(target), mp(-1 * num.first, 0));

    if(it != target.end())
        res.push_back(mp(myabs(it->first + num.first), it->second + num.second));

    if(it != target.begin())
        res.push_back(mp(myabs((it-1)->first + num.first), (it-1)->second + num.second));
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    while (cin >> N, N) {
        vector<ll> group1, group2;
        rep(i, N) {
            ll tmp;
            cin >> tmp;
            if(i <= N/2)
                group1.push_back(tmp);
            else
                group2.push_back(tmp);
        }

        vector< pair<ll, int> > pset1, pset2;
        power_set(group1, pset1);
        power_set(group2, pset2);

        vector< pair<ll, int> > res;
        find_near(pset1, mp(0,0), res);
        find_near(pset2, mp(0,0), res);
        tr(it, pset1)
            find_near(pset2, *it, res);

        sort(all(res));
        cout << res[0].first << " " << res[0].second << endl;
    }

    return 0;
}
```

## POJ2549 - Sumsets
蟻本とおおよそ同じ。
"find the largest d" のlargestを見逃してて投げまくってしまった。

```C++
struct Numpair {
    int sum, one, two;
    bool operator<(const Numpair& rhs) const {
        return sum < rhs.sum;
    }
};
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n;
    while(cin >> n, n) {
        vi nums(n);
        rep(i,n) cin >> nums[i];

        vector<Numpair> ab, cd;
        rep(i, n-1) {
            REP(j, i+1, n) {
                ab.push_back(Numpair{nums[i] + nums[j], i, j});
            }
        }
        rep(i, n) {
            rep(j, n) {
                if(i != j)
                    cd.push_back(Numpair{nums[j] - nums[i], i, j});
            }
        }

        sort(all(cd));

        const int MINF = -536870912 -1;
        int res = MINF;
        tr(ab_it, ab) {
            Numpair base = Numpair{ab_it->sum, 0,0};
            vector<Numpair>::iterator b = lower_bound(all(cd), base);
            vector<Numpair>::iterator e = upper_bound(all(cd), base);

            for(; b != e; ++b) {
                if(b->sum == ab_it->sum) {
                    vi tmp;  // check whether the result uses the same number
                    tmp.push_back(ab_it->one);
                    tmp.push_back(ab_it->two);
                    tmp.push_back(b->one);
                    tmp.push_back(b->two);
                    int tmp_n = tmp.size();
                    sort(all(tmp));
                    tmp.erase( unique(all(tmp)), tmp.end() );
                    int uni_n = tmp.size();
                    if(tmp_n == uni_n) {
                        res = max(res, nums[b->two]);
                    }
                }
            }
        }

        if(res == MINF)
            cout << "no solution" << endl;
        else
            cout << res << endl;
    }

    return 0;
}
```


## AOJ0531 - Paint Color
ほぼ蟻本通り。
線の太さは異なるので、1を引いたりして調整しないといけない。

```C++
int compress(vector<pii>& p, int w) {
    vi xs;
    rep(i, p.size()) {
        REP(d, -1, 1+1) {  // -1, 0, 1  // 左右を含めて保存.
            int tx1 = p[i].first + d, tx2 = p[i].second + d;
            if(0 <= tx1 && tx1 <= w) xs.push_back(tx1);
            if(0 <= tx2 && tx2 <= w) xs.push_back(tx2);
        }
    }

    // 重複を削除.
    sort(all(xs));
    xs.erase(unique(all(xs)), xs.end());

    // 圧縮した値に書き換え.
    rep(i, p.size()) {
        p[i].first = find(xs.begin(), xs.end(), p[i].first) - xs.begin();
        p[i].second = find(xs.begin(), xs.end(), p[i].second) - xs.begin();
    }

    return xs.size();
}
int dx[] = {1, 0, -1, 0};
int dy[] = {0, -1, 0, 1};
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int w, h, n;
    while( cin >> w >> h, (w|h) ) {
        cin >> n;

        vector<pii> px(n), py(n);
        rep(i, n)
            cin >> px[i].first >> py[i].first >> px[i].second >> py[i].second;

        w = compress(px, w);
        h = compress(py, h);

        w--, h--;

        // 塗りつぶし.
        vector<vector<bool> > fld(h, vector<bool>(w, false));
        rep(i, n) {
            REP(y, py[i].first, py[i].second) {
                REP(x, px[i].first, px[i].second)
                    fld[y][x] = true;
            }
        }

        int res = 0;
        rep(r, h) {
            rep(c, w) {
                if (!fld[r][c]) {
                    res++;

                    // bfs
                    queue<pii> que;
                    que.push(mp(r, c));
                    while (!que.empty()) {
                        pii now = que.front();
                        que.pop();

                        rep(d, 4) {
                            int my = now.first + dy[d];
                            int mx = now.second + dx[d];
                            if (0 <= my && my < h && 0 <= mx && mx < w &&
                                !fld[my][mx]) {
                                fld[my][mx] = true;
                                que.push(mp(my, mx));
                            }
                        }
                    }
                }
            }
        }

        cout << res << endl;
    }

    return 0;
}
```


## まとめ
蟻本通りの問題もあったが、全体的に重たかった。
普段慣れない実装が多く、POJはしょうもないミスも発見するのが困難であったため、思ったより時間が掛かってしまった。

この回のタイトルは「頻出テクニック」なので、コンテストでも「これ蟻本で見たことある！」という風になってほしい。
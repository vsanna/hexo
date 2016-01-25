title: "Codeforces #340 Div.2 A-D"
date: 2016-01-26 00:23:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Elephant
[問題文](http://codeforces.com/contest/617/problem/A)

### 考え方
一見パターン数を求める問題に見えるが、単に最短移動回数を求めるだけ。
DPを使うまでもなく、割り算と余りがあるかだけでよい。

```C++
int main() {
    int n;
    cin >> n;
    int t = (n%5 != 0);
    cout << n/5 + t << endl;
    return 0;
}
```

## B. Chocolate
[問題文](http://codeforces.com/contest/617/problem/B)

### 考え方
まず、端っこの0は関係ないので除外。
1の間にある0をカウントし、それぞれの個数を掛けあわせればパターン数になる。
$n$が100までなので、32bitでは収まらないのに注意。
プレテストではオーバーフローするケースは含まれていないらしく、Hackポイントを稼げた。

```C++
int main() {
	int n;
	cin >> n;
	vi a(n);
    rep(i, n)
        cin >> a[i];

    int s=n, e=0;
    rep(i, n) {
        if(a[i] == 1) {
            s = i;
            break;
        }
    }
    for(int i=n-1; i>=0; --i) {
        if(a[i] == 1) {
            e = i;
            break;
        }
    }

    // no nuts
    if(s > e) {
        cout << 0 << endl;
        return 0;
    }
    vi c;
    REP(i, s, e+1) {
        if(a[i] == 0)
            c[c.size()-1]++;
        else
            c.push_back(1);
    }

    ll ans = 1;
    tr(it, c)
        ans *= *it;

    cout << ans << endl;
	
    return 0;
}
```

## C. Watering Flowers
[問題文](http://codeforces.com/contest/617/problem/C)

### 考え方
まず、花の無い位置まで散布する意味は無いため、花の数がループ回数の基準になる。
各花に対し、必要な$r_1$と$r_2$を計算しペアで保持する。
必要な$r_1$が小さい順にソートし、前から順にループして$r_1$とその残りの中で必要な$r_2$の最大を求めればよい。

残りの中での最大を高速に求めるには、後ろからループし$r_2$の最大を保持した配列を作っておけば$O(1)$で求まる。
全体では$O(nlogn)$となる。
（C問題にしては制約ゆるすぎでは……？$O(n^2)$でも通る）

```C++
int main() {
	int n, x1, y1, x2, y2;
	cin >> n >> x1 >> y1 >> x2 >> y2;

    vector<pll> dist(n);
    rep(i,n) {
        ll x, y;
        cin >> x >> y;
        ll d1 = calc_dist(mp(x, y), mp(x1, y1));
        ll d2 = calc_dist(mp(x, y), mp(x2, y2));
        dist[i] = mp(d1, d2);
    }
    sort(all(dist));

    vll mx(n+1, 0);
    for(int i=n-1; i>=0; i--) {
        mx[i] = max(mx[i+1], dist[i].second);
    }
    ll ans = mx[0];

    ll now = 0;
    rep(i, n) {
        chmax(now, dist[i].first);
        chmin(ans, now + mx[i+1]);
    }
    cout << ans << endl;
	
    return 0;
}
```

## D. Polyline
[問題文](http://codeforces.com/contest/617/problem/D)

### 考え方
3点が与えられるので、様々なパターンを考える必要がある。
とは言え、注意すべきなのは↓のケースくらい。
```
・
  ・
・
```
というようなパターンは3つ必要で、これを考慮していない人が多い＆プレテストに無いことでHack祭りになっていた。

```C++
void pc(vector<pii> &p) {
    int y0, y1, y2;
    if (p[0].first == p[1].first)  // 右に凸
        y0 = p[0].second, y1 = p[1].second, y2 = p[2].second;
    else
        y0 = p[1].second, y1 = p[2].second, y2 = p[0].second;

    if (min(y0, y1) < y2 && y2 < max(y0, y1))
        cout << 3 << endl;
    else
        cout << 2 << endl;
}

int main() {
    vector<pii> xpoints(3), ypoints(3);
    map<int, int> xs, ys;
    rep(i, 3) {
        int x, y;
        cin >> x >> y;
        xpoints[i] = mp(x, y);
        ypoints[i] = mp(y, x);
        xs[x]++;
        ys[y]++;
    }
    sort(all(xpoints));
    sort(all(ypoints));

    if (xs.size() == 1 || ys.size() == 1)
        cout << 1 << endl;
    else if (xs.size() == 2)
        pc(xpoints);
    else if (ys.size() == 2)
        pc(ypoints);
    else
        cout << 3 << endl;

    return 0;
}
```


## まとめ
参加者は3646人。

- A: 3514/3604
- B: 2116/3025
- C: 569/1862
- D: 606/1987
- E: 36/101

A-Dは今までにない簡単な問題セットだった。
Hackに大きく左右される不安定なコンテストになった。

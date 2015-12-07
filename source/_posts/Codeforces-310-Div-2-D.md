title: "Codeforces #310 Div.2 D"
date: 2015-12-09 12:10:05
tags:
- esplo77
- Codeforces
- C++11
---

# D. Case of Fugitive


## 問題
[問題文](http://codeforces.com/contest/556/problem/D)
l_i, r_iの区間に島がある。
また、長さがバラバラな橋がある。
隣り合った島に橋を掛けるとき、使う橋の番号を順に出力せよ。

島の数n: $2 \leq n \leq 2\times10^5$
橋の数m: $1 \leq n \leq 2\times10^5$
位置: $1 \leq l,r \leq 10^{18}$


## 考え方
最大幅が大きい順にソートして、短い橋から順番に使うと上手くいく……かと思いきや、色々なケースがあって落ちる。

以下、橋を短い順に取る場合に考えられるケース。
島の区間は(最小幅, 最大幅)で表す。

最小幅でソートするとダメな場合
(1,5), (2,4)
橋: 4, 5

最大幅でソートするとダメな場合
(4, 5), (1, 6)
橋: 1, 4

一度使わなかった橋は使えない（$O(m^2)$になってTLEする）ため、色々なパターンで引っかかる。

### 典型処理
このような場合の最適な割り当て方は、典型的な処理があるようだ。

1. 区間の左端と橋を同じ配列に入れ、小さい順に取り出す。
1. 区間を取り出したら、区間の右端をsetに入れる。
1. 橋を取り出したら、setから最小のものを取りだし割り当てる。

このような処理で、最適な割り当てが行える。
蟻本の練習問題、priority_queueのでもあった。

[参考URL](http://kmjp.hatenablog.jp/entry/2015/07/08/0930)

参考URLの方法が綺麗だった。
インデックスに大きな値を足して、種類の判別とソートの順番変更を同時に行うという工夫。
自分は構造体で無理やり書いてみた。
通るが美しくはない。

```C++
struct Target {
    ll pos;
    bool isBridge;
    int ind;

    // 区間を先に持ってくる
    bool operator<( const Target& right ) const {
        // 面倒なのでpairに入れてソート
        auto t1 = mp(pos, mp(isBridge, ind));
        auto t2 = mp(right.pos, mp(right.isBridge, right.ind));
        return t1 < t2;
    }
};

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int n,m;
    cin >> n >> m;

    vector< pair<ll, ll> > isle(n);
    rep(i, n)
        cin >> isle[i].first >> isle[i].second;

    // 各位置の要素を保持する
    set<Target> targets;

    vector<ll> bridges(m);
    rep(i, m) {
        cin >> bridges[i];
        targets.insert( {bridges[i], true, i} );
    }

    // (min, max)
    vector< pair<ll, ll> > dist(n-1);
    rep(i,n-1) {
        dist[i].first = isle[i+1].first - isle[i].second;
        dist[i].second = isle[i+1].second - isle[i].first;
        targets.insert( {dist[i].first, false, i} );
    }

    vi res(n-1);

    set< pair<ll, int> > candidates;
    tr(it, targets) {
        int ind = it->ind;
        // 区間なら終点を入れて終わり.
        if(! it->isBridge) {
            candidates.insert( mp(dist[ind].second, ind));
            continue;
        }

        // 橋なら使えるかどうか試す

        // 使える候補の区間がなければ何もしない
        if(candidates.empty())
            continue;

        auto s = candidates.begin();

        // 橋が大きすぎて使えない
        if(bridges[ind] > s->first) {
            cout << "No" << endl;
            return 0;
        }

        // 使った
        res[s->second] = ind + 1;
        candidates.erase(s);
    }

    if(!candidates.empty()) {
        cout << "No" << endl;
        return 0;
    }

    cout << "Yes" << endl;
    rep(i, n-1) {
        cout << res[i];
        if(i == n - 2)
            cout << endl;
        else
            cout << " ";
    }

    return 0;
}
```

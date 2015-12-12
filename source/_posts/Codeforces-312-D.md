title: "Codeforces #312 D"
date: 2015-12-13 02:26:01
tags:
- esplo77
- Codeforces
- C++11
---

# D. Guess Your Way Out! II
[問題文](http://codeforces.com/contest/558/problem/D)

## 考え方
最初に、クエリは最下層でのインデックスに変換する。
その後、クエリを処理してゆく。

最初は全範囲が答えの候補。
有効な範囲を示すクエリは、候補から積集合を取ると処理できる。
無効な範囲を示すクエリは、順番に見てゆき候補から除外してゆく。

候補から除外するために、

- 左端が小さいクエリから順に処理し、候補の左端を狭めて（右にずらして）ゆく
- 右端が大きいクエリから順に処理し、候補の右端を狭めて（左にずらして）ゆく

という2つの処理を行い、最終的に候補の左端と右端が一致すれば正しいデータ、候補が無ければチート、それ以外はデータ不足。

## 回答

```C++
int h;

ll leftmost(int level, ll v) {
    int rem = h - level;
    ll res = v;
    while(rem--)
        res *= 2;
    return res;
}
ll rightmost(int level, ll v) {
    int rem = h - level;
    ll res = v;
    while(rem--)
        res *= 2, res+=1;
    return res;
}
pair<ll, ll> intersect( pair<ll,ll> use, pair<ll,ll> area ) {
    pair<ll, ll> left = min(use, area),
            right = max(use, area);
    if(right.first <= left.second)
        return mp(max(left.first, right.first), min(left.second, right.second));
    else
        return mp(-1, -1);
}
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int q;
    cin >> h >> q;

    vector< pair<ll,ll> > unuseL, unuseR;
    pair<ll,ll> use = mp(leftmost(1,1), rightmost(1,1));

    // process 'use' first
    rep(i, q) {
        int level;
        ll L, R;
        int ans;
        cin >> level >> L >> R >> ans;
        pair<ll, ll> area = mp(leftmost(level, L), rightmost(level, R));
        if(ans == 0) {
            unuseL.push_back(area);  // 左端を決める用
            swap(area.first, area.second);
            unuseR.push_back(area);
        }
        else
            use = intersect(use, area);
    }
    if(use == mp((ll)-1, (ll)-1)) {
        cout << "Game cheated!" << endl;
        return 0;
    }

    // process 'unuse'
    ll left = use.first, right = use.second;

    sort(all(unuseL));
    sort(all(unuseR));
    reverse(all(unuseR));  // 右端が大きい順

    // 除外していって左端を決める
    tr(it, unuseL) {
        // 間に入っていたら除外
        if(it->first <= left && left <= it->second)
            left = it->second + 1;
    }
    // 右端も
    tr(it, unuseR) {
        if(it->second <= right && right <= it->first)
            right = it->second - 1;
    }

    if(left == right)
        cout << left << endl;
    else if(left > right)
        cout << "Game cheated!" << endl;
    else
        cout << "Data not sufficient!" << endl;

    return 0;
}
```

## まとめ
積集合を取る箇所は、単純にmin/maxで問題ないが、面倒な処理を書いてしまった。
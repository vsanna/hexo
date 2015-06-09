title: "Codeforces #303 (Div. 2) D"
date: 2015-05-24 17:52:55
tags:
- esplo77
- Codeforces
- C++11
---

# D. Queue

D...？

## 問題

n人が並んでおり、それぞれにサービスの所要時間が設定されている。
サービスの所要時間を待機時間が上回った場合、お客は不満を持ってしまう。
待機時間は、その人の前に並んでいる人たちのサービス所要時間を合計した値である。
人を上手く並び替えて、不満を持たない人が最大になるようにしたとき、その人数を出力せよ。

$$
1 \leq n \leq 10^5 \\\
1 \leq t_i \leq 10^9
$$

## 考え方

例えば、2, 4, 5, 7, 9という列を考える。
この場合、2, 4, 7, 5, 9で最初の3人が満足するのが最大。
ここで、2, 4, 9, 5, 7とすることもできるが、デメリットしかない。
条件を満たすお客が複数いる場合は、所要時間が一番小さいお客を取ってくるのが最も良いことは明らか。

というわけで、所要時間が小さい人から順番に取り出し、条件を満たせば列に追加、余りは列の最後に適当に置いておけば、満足する人が最大な列が出来上がる。

普通に配列をsortしてもいいし、C++であればpriority_queueを使えば最速。

## コード
```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    priority_queue<int, vector<int>, greater<int> > que;
    int n;
    cin >> n;
    rep(i,n) {
        int t;
        cin >> t;
        que.push(t);
    }

    int res = 0;
    ll tmp = 0ll;
    rep(i, n) {
        int t = que.top(); que.pop();
        if(tmp <= t) {
            tmp += t;
            res++;
        }
    }

    cout << res << endl;

    return 0;
}
```

## 結果
93ms, 1044KB


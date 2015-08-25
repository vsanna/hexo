title: "Ari-practice solve #4"
date: 2015-08-17 23:18:00
tags:
- esplo77
- C++
- 蟻本
---

# 蟻本練習問題を解いた (4)
貪欲に。

## POJ2376 - Cleaning Shifts
貪欲。
微妙にややこしい場合分けがあるので、問題をちゃんと読んでテストケースをちゃんと用意した方がいいかも。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, T;
    cin >> N >> T;

    priority_queue<pii, vector<pii>, greater<pii> > que;
    rep(i,N) {
        int t1, t2;
        cin >> t1 >> t2;
        que.push(mp(t1,t2));
    }

    int next_low = 1;
    int next_max = 1;
    int res = 0;
    while(!que.empty()) {
        pii t = que.top(); que.pop();
        if(t.first <= next_low)
            next_max = max(next_max, t.second + 1);
        else if(t.first <= next_max) {
            next_low = next_max;
            que.push(t);
            res++;
        }
        else {
            cout << -1 << endl;
            return 0;
        }

        if(next_max > T) {
            cout << res + 1 << endl;
            return 0;
        }
    }

    cout << -1 << endl;
    return 0;
}
```

## POJ1328 - Radar Installation
左から見ていくだけ。
なのになぜか通らない……。
諦め。せめて何個のテストケースに通ったのかが知りたい……。


## POJ3190 - Stall Reservations
やることは簡単で、1つが終了したらどけ、その隙間に次のを配置するだけ。
ただデータ構造が結構複雑になりがちなので注意が必要。
構造体を使って少しでもシンプルにしたいところ。

```C++

struct Range {
    int from;
    int to;
    int index;
};
bool cmpFrom (const Range& lhs, const Range& rhs) {
    return lhs.from < rhs.from;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    vector<Range> lr(N);
    priority_queue<pii, vector<pii>, greater<pii> > stall;

    rep(i, N) {
        Range r;
        cin >> r.from >> r.to;
        r.index = i;
        lr[i] = r;
    }

    sort(all(lr), cmpFrom);

    vi res(N);

    rep(i, N) {
        // new stall
        if ( stall.empty() || stall.top().first >= lr[i].from ){
            stall.push(mp(lr[i].to, stall.size()+1));
            res[lr[i].index] = stall.size();
        }
        else {
            pii t = stall.top(); stall.pop();
            res[lr[i].index] = t.second;
            stall.push(mp(lr[i].to, t.second));
        }
    }

    cout << stall.size() << endl;
    tr(it, res) {
        cout << *it << endl;
    }

    return 0;
}
```

## POJ2393 - Yogurt factory
単純な貪欲。
[他の人の回答](http://d.hatena.ne.jp/atetubou/20110504/1304478314)見たら、めちゃくちゃ短かった。
無駄にややこしくしてしまて、修行不足を感じる。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N,S;
    cin >> N >> S;

    vector<pii> yo(N);
    rep(i,N)
        cin >> yo[i].first >> yo[i].second;

    ll res = 0;
    rep(i,N) {
        int nowp = yo[i].first;
        int amount = yo[i].second;
        int start = i;

        int nextp = nowp;

        while(i != N) {
            nextp += S;
            i++;
            if(yo[i].first > nextp ) {
                amount += yo[i].second;
                res += (ll)yo[i].second * (i-start) * S;
            }
            else {
                i--;
                break;
            }
        }
        res += (ll)nowp * amount;
    }

    cout << res << endl;

    return 0;
}
```


## POJ1017 - Packet
スーパー場合分けゲー。
面倒なので省略させていただいた……。

## POJ3040 - Allowance
大きい順に払った後、小さい順に払うという、微妙に証明しにくい感じの貪欲法を使う。
思いつかなかった。

「コインの枚数が0の場合にif文の中を実行しない」というちょっとした高速化をしないと、TLEになった。
時間制限厳しすぎる……。

```C++
int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N, C;
    cin >> N >> C;

    vector<pii> coins(N);
    int res = 0;
    rep(i, N) {
        cin >> coins[i].first >> coins[i].second;
        if(coins[i].first >= C) {
            res += coins[i].second;
            coins[i].second = 0;
        }
    }
    sort(all(coins));

    while(true) {
        int current = 0;

        for(int i=N-1; i>=0; --i) {
            if( coins[i].second > 0 && current + coins[i].first <= C) {
                int num = (C - current) / coins[i].first;
                int use = min(num, coins[i].second);
                current += use * coins[i].first;
                coins[i].second -= use;
            }
        }

        rep(i,N) {
            if( coins[i].second > 0 && current < C) {
                int num = ceil( (double)(C - current) / coins[i].first);
                int use = min(num, coins[i].second);
                current += use * coins[i].first;
                coins[i].second -= use;
            }
        }

        if(current < C)
            break;

        res++;
    }

    cout << res << endl;

    return 0;
}
```


## POJ1862 - Stripies
なるべくルートを活用したいので、大きい順の貪欲。

```C++

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(3);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    deque<double> a;
    rep(i,N){
        int t;
        cin >> t;
        a.push_back(t);
    }

    while(true) {
        if(a.size() == 1)
            break;

        sort(all(a));

        double f = a.back(); a.pop_back();
        double b = a.back(); a.pop_back();

        a.push_back(2.0 * sqrt(f * b));
    }

    cout << a.front() << endl;

    return 0;
}
```

## POJ3262 - Protecting the Flowers
式を立てると論理的に解ける。
あんまり見たことのない感じだったので、ためになった。

```C++
bool asc( const pii& left, const pii& right ) {
    // D * T
    return right.first * left.second < left.first * right.second;
}

int main() {
    cout.setf(ios::fixed, ios::floatfield);
    cout.precision(8);
    ios_base::sync_with_stdio(false);

    int N;
    cin >> N;

    int dsum = 0;
    vector<pii> d_t(N);
    rep(i,N) {
        cin >> d_t[i].second >> d_t[i].first;
        dsum += d_t[i].first;
    }

    sort(all(d_t), asc);

    ll res = 0;
    rep(i,N) {
        dsum -= d_t[i].first;
        res += dsum * (d_t[i].second*2);
    }
    cout << res << endl;

    return 0;
}
```

## まとめ
手法は貪欲だが、どう貪欲を使うのかが結構難しい。
この辺を鍛え上げるとコドフォのC問題とかで役に立ちそうな気がする。

title: "Codeforces #342 A-C"
date: 2016-02-25 23:39:22
tags:
- esplo77
- Codeforces
- C++11
---

## A. Guest From the Past
[問題文](http://codeforces.com/contest/625/problem/A)

### 考え方
$a <= b - c$であれば、aのみを限界まで利用すれば良い。
それ以外であれば、なるべく$b-c$を利用し、$b$を払えない段階になったらaに切り替えれば良い。

通常のA問題であれば愚直にシミュレートできるが、この問題では制約が$10^{18}$と大きく、ループすると様々な所でTLEする。
そのため、$\frac{n-b}{b-c}$として、$b-c$が使える回数を計算する。
$n-b$が負になる際は事前に場合分けしておくと計算しやすい。

### コード

```C++
int main() {
	ll n, a, b, c;
    cin >> n >> a >> b >> c;

    if(a <= (b-c)) {
        cout << n / a << endl;
        return 0;
    }

    if(n >= b) {
        ll x = (n - b) / (b - c);
        x++;

        ll rem = n - x * (b - c);
        cout << x + rem / a << endl;
    }
    else {
        cout << n / a << endl;
    }

    return 0;
}
```


## B. War of the Corporations
[問題文](http://codeforces.com/contest/625/problem/B)

### 考え方
ある文字列s（$|s| \leq 10^5$）に別の文字列t（$|t| \leq 200$）が含まれないようにするための最小変更文字数を求める。
先頭からt文字ずつチェックをしてゆき、一致した文字を読み飛ばしてカウントを再開すれば良い。
$O(|s||t|)$

コンテスト中は、なぜかsのプレフィックスを全て求め、一致した部分文字列の末尾を書き換えるという無駄なコードを書いていた。

### コード

```C++
int main() {
	string g, a;
    cin >> g >> a;

    vector<string> prefs;
    rep(i, g.size())
        prefs.push_back(g.substr(i, a.size()));

    ll ans = 0;
    rep(i, prefs.size()) {
        if(a == prefs[i]) {
            ans++;
            REP(j, 1, a.size()) {
                if(i+j >= prefs.size()) break;
                prefs[i+j][a.size() - j - 1] = '#';
            }
        }
    }

    cout << ans << endl;
	
    return 0;
}
```


## C. K-special Tables
[問題文](http://codeforces.com/contest/625/problem/C)

### 考え方
指定された列が最大になるためには、値が大きい順に行列の右下からk列目まで左に埋めてゆき、1つ上の行の右橋からまた埋めてゆけば良い。
埋め終わったら、今度は左上から小さい値を順に埋めてゆけば、答えの行列が出来上がる。

↓ではなるべくミスを減らしたかったので、priority_queueに詰めたり別のpriority_queueに詰めなおしたりとややこしいことをしている。

### コード

```C++
int main() {
    int n, k;
    cin >> n >> k;
    k--;
    vvi mat(n, vi(n, -1));

    priority_queue<int> que;
    rep(i, n*n) {
        que.push(i+1);
    }

    for(int r=n-1; r>=0;r--) {
        for(int c=n-1; c>=k; c--) {
            int q = que.top(); que.pop();
            mat[r][c] = q;
        }
    }

    priority_queue <int, vi, greater<int> > nums;
    while(!que.empty()) {
        int q = que.top(); que.pop();
        nums.push(q);
    }

    rep(r, n) {
        rep(c, n) {
            if(mat[r][c] == -1) {
                int q = nums.top(); nums.pop();
                mat[r][c] = q;
            }
        }
    }

    ll sum = 0;
    rep(i, n) {
        sum += mat[i][k];
    }
    cout << sum << endl;

    rep(i, n) {
        rep(j, n) {
            cout << mat[i][j] << " ";
        }
        cout << endl;
    }
	
    return 0;
}
```


## まとめ
参加者は2756人。

- A: 579/2586
- B: 1722/2362
- C: 1781/1834
- D: 5/257
- E: 0/5

A大虐殺。
Dも普段のE並の正答率で、ルーム運に左右されやすいセットだった。
